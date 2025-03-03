# Cartera TMobilitat - Implementation Guide

## Table of Contents
1. [Overview](#overview)
2. [Project Setup](#project-setup)
3. [Pass Structure](#tmobilitat-pass-structure)
4. [Pass Generation](#1-pass-generation)
5. [NFC Validation](#2-nfc-validation)
6. [Pass Updates](#3-pass-updates)
7. [User Interface](#4-user-interface)
8. [Error Handling](#error-handling)
9. [Analytics](#analytics)
10. [Testing Guide](#testing-guide)
11. [NFC Setup Details](#nfc-setup-details)
12. [Update System Details](#update-system-details)

## Overview

This guide covers the implementation of Apple Wallet functionality for the Cartera TMobilitat app, allowing users to:
1. Add their TMobilitat pass to Apple Wallet
2. Validate passes using NFC
3. Receive automatic pass updates
4. View pass status and history

## Project Setup

### Required Capabilities
Add to your Xcode project:
```
Target > Signing & Capabilities:
1. Wallet
2. Near Field Communication Tag Reading
3. Push Notifications
```

### Entitlements Configuration
```xml
<dict>
    <key>com.apple.developer.pass-type-identifiers</key>
    <array>
        <string>$(TeamIdentifierPrefix)pass.tmobilitat.cartera</string>
    </array>
    <key>com.apple.developer.nfc.readersession.formats</key>
    <array>
        <string>NDEF</string>
        <string>TAG</string>
    </array>
</dict>
```

### Info.plist Settings
```xml
<key>NFCReaderUsageDescription</key>
<string>Necessitem accés NFC per validar el teu títol de transport</string>
```

## TMobilitat Pass Structure

### Pass Package
```
TMobilitat.pass/
├── pass.json       # Pass data and configuration
├── manifest.json   # File hashes
├── signature      # Pass signature
├── logo.png       # TMobilitat logo
├── logo@2x.png    # Retina logo
├── icon.png       # App icon
└── icon@2x.png    # Retina icon
```

### Pass Data Structure
```swift
struct TMobilitatPass {
    let passTypeIdentifier = "pass.tmobilitat.cartera"
    let serialNumber: String      // Unique pass identifier
    let userIdentifier: String    // TMobilitat user ID
    let passProduct: String       // Pass type (T-usual, T-casual, etc.)
    let validFrom: Date          // Start date
    let validUntil: Date        // Expiration date
    let zonesValid: [String]    // Valid zones
    let remainingTrips: Int?    // For T-casual
}
```

## Implementation

### 1. Pass Generation

```swift
class TMobilitatPassGenerator {
    static func createPass(
        userID: String,
        product: TMobilitatProduct,
        validFrom: Date,
        validUntil: Date,
        zones: [String]
    ) throws -> PKPass {
        let passData: [String: Any] = [
            "formatVersion": 1,
            "passTypeIdentifier": "pass.tmobilitat.cartera",
            "serialNumber": generateSerialNumber(userID: userID),
            "teamIdentifier": "YOUR_TEAM_ID",
            "organizationName": "TMobilitat",
            "description": product.description,
            
            // Visual appearance
            "backgroundColor": "rgb(0, 155, 216)", // TMobilitat blue
            "foregroundColor": "rgb(255, 255, 255)",
            "labelColor": "rgb(255, 255, 255)",
            
            // Pass data
            "generic": [
                "primaryFields": [
                    [
                        "key": "product",
                        "label": "TÍTOL",
                        "value": product.name
                    ]
                ],
                "secondaryFields": [
                    [
                        "key": "validFrom",
                        "label": "VÀLID DES DE",
                        "value": formatDate(validFrom)
                    ],
                    [
                        "key": "validUntil",
                        "label": "VÀLID FINS",
                        "value": formatDate(validUntil)
                    ]
                ],
                "auxiliaryFields": [
                    [
                        "key": "zones",
                        "label": "ZONES",
                        "value": zones.joined(separator: ", ")
                    ]
                ]
            ],
            
            // NFC configuration
            "nfc": [
                "message": "Apropa el mòbil al validador",
                "encryptionPublicKey": loadEncryptionKey(),
                "requiresAuthentication": true
            ]
        ]
        
        return try generateSignedPass(with: passData)
    }
}
```

### 2. NFC Validation

```swift
class TMobilitatValidator: NSObject, NFCNDEFReaderSessionDelegate {
    private var session: NFCNDEFReaderSession?
    
    func validatePass(_ pass: PKPass) {
        session = NFCNDEFReaderSession(
            delegate: self,
            queue: .main,
            invalidateAfterFirstRead: true
        )
        
        session?.alertMessage = "Apropa el mòbil al validador"
        session?.begin()
    }
    
    func readerSession(
        _ session: NFCNDEFReaderSession,
        didDetectNDEFs messages: [NFCNDEFMessage]
    ) {
        guard let validatorData = extractValidatorData(from: messages.first)
        else {
            session.invalidate(errorMessage: "Error de lectura")
            return
        }
        
        validateWithServer(validatorData) { result in
            switch result {
            case .success:
                session.alertMessage = "Títol validat correctament"
                NotificationCenter.default.post(
                    name: .passValidated,
                    object: nil
                )
            case .failure(let error):
                session.invalidate(errorMessage: error.localizedDescription)
            }
        }
    }
}
```

### 3. Pass Updates

#### Overview

This section outlines the process for implementing and managing updates to TMobilitat passes in Apple Wallet. It covers the technical implementation of the pass update system, including server-side components and client-side handling.

#### Basic Setup

First, add these to your pass JSON:
```json
{
    "webServiceURL": "https://api.tmobilitat.cat/passes/",
    "authenticationToken": "your-secret-token",
    "serialNumber": "unique-pass-id"
}
```

#### Pass Update Manager Implementation

Here's a basic update manager:

```swift
import PassKit

class TMobilitatPassUpdateManager {
    // Keep track of passes
    static let shared = TMobilitatPassUpdateManager()
    private var passes: [PKPass] = []
    
    // Register a pass for updates
    func registerPass(_ pass: PKPass) {
        passes.append(pass)
        
        // Tell Apple's servers about it
        PKPassLibrary().addPasses([pass]) { error in
            if let error = error {
                print("No s'ha pogut afegir el títol: \(error.localizedDescription)")
                return
            }
            print("Títol afegit correctament!")
        }
    }
    
    // Check for updates
    func checkForUpdates() {
        passes.forEach { pass in
            checkPassUpdate(pass)
        }
    }
    
    // Check one pass
    private func checkPassUpdate(_ pass: PKPass) {
        let url = URL(string: "https://api.tmobilitat.cat/passes/\(pass.serialNumber)")!
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let data = data {
                self.handleUpdateResponse(data, for: pass)
            }
        }.resume()
    }
    
    // Handle the server's response
    private func handleUpdateResponse(_ data: Data, for pass: PKPass) {
        // Update the pass if needed
        if needsUpdate(data) {
            updatePass(pass)
        }
    }
    
    // Simple update check
    private func needsUpdate(_ data: Data) -> Bool {
        // Add your update logic here
        return true
    }
    
    // Update the pass
    private func updatePass(_ pass: PKPass) {
        // Get new pass data from your server
        // Then update it in Wallet
        print("Actualitzant títol...")
    }
}
```

#### Adding Push Notifications

To send updates automatically:

```swift
class TMobilitatPushNotificationManager: NSObject, PKPushRegistryDelegate {
    let pushRegistry = PKPushRegistry(queue: .main)
    
    override init() {
        super.init()
        setupPushNotifications()
    }
    
    func setupPushNotifications() {
        // Set up push notifications
        pushRegistry.delegate = self
        pushRegistry.desiredPushTypes = [.pkPasses]
    }
    
    // When we get a push notification
    func pushRegistry(
        _ registry: PKPushRegistry,
        didReceiveIncomingPushWith payload: PKPushPayload,
        for type: PKPushType
    ) {
        // Handle the push notification
        print("Notificació rebuda!")
        TMobilitatPassUpdateManager.shared.checkForUpdates()
    }
}
```

#### Server-Side Implementation

Here's what your server needs to do:

```swift
// This would be in your server code (shown in Swift, but use any language!)
func handlePassUpdate(request: Request) -> Response {
    // Get the pass serial number
    let serialNumber = request.serialNumber
    
    // Check if pass needs update
    if passNeedsUpdate(serialNumber) {
        // Create new pass data
        let newPassData = createUpdatedPass(serialNumber)
        
        // Send push notification
        sendPushNotification(serialNumber)
        
        return .success(newPassData)
    }
    
    return .notModified
}
```

#### Testing Updates

1. **Test without Push:**
```swift
// In your app
TMobilitatPassUpdateManager.shared.checkForUpdates()
```

2. **Test with Push:**
- Use Apple's Push Notification Tool
- Or send a test push from your server

#### Update UI Enhancements

1. **Add an Update Animation:**
```swift
func showUpdateAnimation() {
    let animation = CABasicAnimation(keyPath: "transform.rotation")
    animation.fromValue = 0
    animation.toValue = 2 * Double.pi
    animation.duration = 1
    
    updateIcon.layer.add(animation, forKey: "rotation")
}
```

2. **Show Update Progress:**
```swift
func showUpdateProgress() {
    let progress = UIProgressView(progressViewStyle: .default)
    progress.progress = 0
    
    UIView.animate(withDuration: 1) {
        progress.progress = 1
    }
}
```

#### Troubleshooting Updates

##### Common Issues and Solutions

1. **Updates aren't working**
   - Check your server URL is correct
   - Make sure push notifications are enabled
   - Verify your authentication token

2. **Push notifications aren't coming through**
   - Check your certificates
   - Verify your server is sending pushes correctly
   - Make sure the device is registered

##### Advanced Update Features

1. **Automatic Updates**
```swift
Timer.scheduledTimer(
    withTimeInterval: 3600, // Check every hour
    repeats: true
) { _ in
    TMobilitatPassUpdateManager.shared.checkForUpdates()
}
```

2. **Update History**
```swift
class TMobilitatUpdateHistory {
    static var updates: [Update] = []
    
    static func logUpdate(_ update: Update) {
        updates.append(update)
        saveHistory()
    }
}
```

3. **Smart Updates**
```swift
func shouldUpdate(_ pass: PKPass) -> Bool {
    // Only update if:
    // 1. Pass is about to expire
    // 2. Information has changed
    // 3. Status has changed
    return pass.isAboutToExpire ||
           pass.hasNewInformation ||
           pass.statusChanged
}
```

### 4. User Interface

```swift
class PassViewController: UIViewController {
    private let passManager = TMobilitatPassManager.shared
    private let validator = TMobilitatValidator()
    
    @IBAction func addToWalletTapped() {
        guard let userPass = getCurrentUserPass() else { return }
        
        do {
            let pass = try TMobilitatPassGenerator.createPass(
                userID: userPass.userID,
                product: userPass.product,
                validFrom: userPass.validFrom,
                validUntil: userPass.validUntil,
                zones: userPass.zones
            )
            
            let passController = PKAddPassesViewController(pass: pass)
            passController.delegate = self
            present(passController, animated: true)
            
        } catch {
            showError(error)
        }
    }
    
    @IBAction func validatePassTapped() {
        guard let pass = getCurrentWalletPass() else {
            showError("Pass not found in wallet")
            return
        }
        
        validator.validatePass(pass)
    }
}
```

## Error Handling

```swift
enum TMobilitatError: LocalizedError {
    case passNotFound
    case invalidPass
    case validationFailed
    case networkError
    case serverError
    
    var errorDescription: String? {
        switch self {
        case .passNotFound:
            return "No s'ha trobat el títol al Wallet"
        case .invalidPass:
            return "El títol no és vàlid"
        case .validationFailed:
            return "Error de validació"
        case .networkError:
            return "Error de connexió"
        case .serverError:
            return "Error del servidor"
        }
    }
}
```

## Analytics

```swift
class TMobilitatAnalytics {
    static func logPassEvent(_ event: PassEvent) {
        Analytics.logEvent(event.rawValue, parameters: [
            "timestamp": Date(),
            "pass_type": event.passType,
            "zones": event.zones,
            "result": event.result
        ])
    }
}

enum PassEvent: String {
    case addedToWallet = "pass_added"
    case validated = "pass_validated"
    case updated = "pass_updated"
    case expired = "pass_expired"
}
```

## Testing Guide

### Test Cases
1. Pass Creation
   - Verify all pass fields are correct
   - Check TMobilitat branding
   - Validate pass signature

2. NFC Validation
   - Test with valid pass
   - Test with expired pass
   - Test with different zones
   - Test offline validation

3. Updates
   - Test push notification reception
   - Verify pass data updates correctly
   - Check expiration handling

### Test Environment
```swift
class TMobilitatTestHelper {
    static func createTestPass() -> PKPass {
        return try! TMobilitatPassGenerator.createPass(
            userID: "TEST_USER",
            product: .tUsual,
            validFrom: Date(),
            validUntil: Date().addingTimeInterval(30 * 24 * 60 * 60),
            zones: ["1"]
        )
    }
    
    static func simulateValidation() {
        // Simulate NFC validation
        NotificationCenter.default.post(
            name: .passValidated,
            object: nil
        )
    }
}
```

## NFC Setup Details

### Enable NFC in Your App

1. Open Xcode
2. Click on your project
3. Select your target
4. Go to "Signing & Capabilities"
5. Click "+" and add "Near Field Communication Tag Reading"

Your `Info.plist` needs this:
```xml
<key>NFCReaderUsageDescription</key>
<string>Necessitem accés NFC per validar el teu títol de transport</string>
```

### Basic NFC Reader Implementation

```swift
import CoreNFC

class PassValidator: NSObject, NFCNDEFReaderSessionDelegate {
    var nfcSession: NFCNDEFReaderSession?
    
    // Start scanning for NFC
    func startScanning() {
        nfcSession = NFCNDEFReaderSession(
            delegate: self,
            queue: DispatchQueue.main,
            invalidateAfterFirstRead: true
        )
        nfcSession?.begin()
    }
    
    // When we find an NFC tag
    func readerSession(
        _ session: NFCNDEFReaderSession,
        didDetect tags: [NFCNDEFTag]
    ) {
        // We found a tag!
        if let tag = tags.first {
            session.connect(to: tag) { error in
                if error != nil {
                    session.invalidate(errorMessage: "Failed to connect to tag")
                    return
                }
                
                // Read the tag
                tag.queryNDEFStatus { status, _, error in
                    if status == .notSupported {
                        session.invalidate(errorMessage: "Tag not supported")
                        return
                    }
                    
                    // Read the message from the tag
                    tag.readNDEF { message, error in
                        if let error = error {
                            session.invalidate(errorMessage: "Error reading tag")
                            return
                        }
                        
                        // We got the data!
                        self.handleTagData(message)
                        session.invalidate()
                    }
                }
            }
        }
    }
    
    // Handle any errors
    func readerSession(
        _ session: NFCNDEFReaderSession,
        didInvalidateWithError error: Error
    ) {
        // Something went wrong
        print("Error reading NFC: \(error.localizedDescription)")
    }
    
    // Process the tag data
    private func handleTagData(_ message: NFCNDEFMessage?) {
        // Handle the data here!
        print("Got NFC data!")
    }
}
```

### Common NFC Problems & Solutions

#### "NFC Not Working!"
1. Check that NFC is enabled in your phone settings
2. Make sure you're using a real device (not simulator)
3. Try restarting your app

#### "Permission Denied!"
1. Check your `Info.plist` has the NFC permission
2. Make sure NFC capability is added in Xcode

#### "Can't Read Tag!"
1. Make sure you're close enough to the tag
2. Try moving your phone around slightly
3. Check if the tag is NFC NDEF formatted

## Update System Details

### Basic Update Manager

```swift
import PassKit

class PassUpdateManager {
    // Keep track of passes
    static let shared = PassUpdateManager()
    private var passes: [PKPass] = []
    
    // Register a pass for updates
    func registerPass(_ pass: PKPass) {
        passes.append(pass)
        
        // Tell Apple's servers about it
        PKPassLibrary().addPasses([pass]) { error in
            if let error = error {
                print("Couldn't add pass: \(error.localizedDescription)")
                return
            }
            print("Pass added successfully!")
        }
    }
    
    // Check for updates
    func checkForUpdates() {
        passes.forEach { pass in
            checkPassUpdate(pass)
        }
    }
    
    // Check one pass
    private func checkPassUpdate(_ pass: PKPass) {
        let url = URL(string: "https://your-server.com/passes/\(pass.serialNumber)")!
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let data = data {
                self.handleUpdateResponse(data, for: pass)
            }
        }.resume()
    }
    
    // Handle the server's response
    private func handleUpdateResponse(_ data: Data, for pass: PKPass) {
        // Update the pass if needed
        if needsUpdate(data) {
            updatePass(pass)
        }
    }
    
    // Simple update check
    private func needsUpdate(_ data: Data) -> Bool {
        // Add your update logic here
        return true
    }
    
    // Update the pass
    private func updatePass(_ pass: PKPass) {
        // Get new pass data from your server
        // Then update it in Wallet
        print("Updating pass...")
    }
}
```

### Push Notifications for Updates

```swift
class PushNotificationManager: NSObject, PKPushRegistryDelegate {
    let pushRegistry = PKPushRegistry(queue: .main)
    
    override init() {
        super.init()
        setupPushNotifications()
    }
    
    func setupPushNotifications() {
        // Set up push notifications
        pushRegistry.delegate = self
        pushRegistry.desiredPushTypes = [.pkPasses]
    }
    
    // When we get a push notification
    func pushRegistry(
        _ registry: PKPushRegistry,
        didReceiveIncomingPushWith payload: PKPushPayload,
        for type: PKPushType
    ) {
        // Handle the push notification
        print("Got a push!")
        PassUpdateManager.shared.checkForUpdates()
    }
}
```

### Server-Side Update Implementation

```swift
// This would be in your server code (shown in Swift, but use any language!)
func handlePassUpdate(request: Request) -> Response {
    // Get the pass serial number
    let serialNumber = request.serialNumber
    
    // Check if pass needs update
    if passNeedsUpdate(serialNumber) {
        // Create new pass data
        let newPassData = createUpdatedPass(serialNumber)
        
        // Send push notification
        sendPushNotification(serialNumber)
        
        return .success(newPassData)
    }
    
    return .notModified
}
```

## NFC Validation Flow

### System Architecture
```
+----------------+     +----------------+     +----------------+
|                |     |                |     |                |
|  Dispositiu iOS|     |  Validador NFC |     |    Servidor   |
|                |     |                |     |                |
+----------------+     +----------------+     +----------------+
        |                     |                      |
        |   1. Iniciar       |                      |
        |    Validació       |                      |
        |------------------>|                      |
        |                   |                      |
        |   2. Llegir Tag   |                      |
        |<------------------|                      |
        |                   |                      |
        |   3. Enviar Dades |                      |
        |---------------------------------------->|
        |                   |                      |
        |   4. Validar      |                      |
        |<----------------------------------------|
        |                   |                      |
        |   5. Mostrar      |                      |
        |    Resultat       |                      |
        |------------------>|                      |
```

### Validation States
```
+---------------+     +--------------+     +---------------+
|               |     |              |     |               |
|   Escaneig    |---->|  Processant  |---->|   Resultat   |
|               |     |              |     |               |
+---------------+     +--------------+     +---------------+

Estats:
1. Escaneig:    Buscant validador NFC
2. Processant:  Validant dades del títol
3. Resultat:    Mostrant resultat de la validació
```

### User Interface Flow
```
+------------------+     +------------------+     +------------------+
|                  |     |                  |     |                  |
|   Toca per       |     |   Escanejant...  |     |     Resultat    |
|   escanejar      |     |   +---------+    |     |   +---------+   |
|   +---------+    |---->|   | ~~~~    |    |---->|   |    o    |   |
|   |         |    |     |   | ~~~~    |    |     |   |    ✗    |   |
|   |    ▼    |    |     |   +---------+    |     |   +---------+   |
|   |         |    |     |                  |     |                  |
+------------------+     +------------------+     +------------------+
```

### Data Flow
```
1. Petició Inicial
+----------------+
| ID Dispositiu  |
| ID Títol       |
| Timestamp      |
+----------------+
        |
        v
2. Lectura NFC
+----------------+
| ID Validador   |
| Dades Títol    |
| Signatura      |
+----------------+
        |
        v
3. Petició Validació
+----------------+
| Dades Títol    |
| Ubicació       |
| Timestamp      |
+----------------+
        |
        v
4. Resposta Servidor
+----------------+
| Vàlid: Sí/No   |
| Codi Error     |
| Missatge       |
+----------------+
```

### Error Handling
```
+-------------------+     +-------------------+
|                   |     |                   |
|   Error Detectat  |     | Missatge Usuari  |
|                   |     |                   |
+-------------------+     +-------------------+
         |                         ^
         |                         |
         v                         |
+-------------------+     +-------------------+
|                   |     |                   |
|  Anàlisi d'Error  |---->|   Acció Error    |
|                   |     |                   |
+-------------------+     +-------------------+

Errors Comuns:
1. NFC Desactivat
2. Error de Lectura
3. Títol Invàlid
4. Error de Xarxa
5. Error de Servidor
```

### Security Flow
```
+------------------+     +------------------+
|                  |     |                  |
| Signatura Títol  |     | Encriptació NFC  |
|                  |     |                  |
+------------------+     +------------------+
         |                       |
         v                       v
+------------------+     +------------------+
|                  |     |                  |
| Verificació      |     |  Desencriptació  |
| Signatura        |     |                  |
+------------------+     +------------------+
         |                       |
         v                       v
+------------------+     +------------------+
|                  |     |                  |
|  Autenticació    |     |    Validació     |
|                  |     |                  |
+------------------+     +------------------+
```

### Implementation Sequence
```
1. Configuració
   +----------------+
   | Configurar NFC |
   +----------------+
          |
          v
2. Escaneig
   +----------------+
   | Iniciar Sessió |
   +----------------+
          |
          v
3. Lectura
   +----------------+
   | Processar Tag  |
   +----------------+
          |
          v
4. Validació
   +----------------+
   | Verificar Dades|
   +----------------+
          |
          v
5. Resposta
   +----------------+
   | Mostrar Result |
   +----------------+
``` 