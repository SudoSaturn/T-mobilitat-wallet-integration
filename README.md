# Cartera TMobilitat - Add to Wallet Integration
![Apple Wallet]([https://developer.apple.com/wallet/add-to-apple-wallet-guidelines/images/wallet-card-icon_2x.png](https://upload.wikimedia.org/wikipedia/commons/thumb/3/30/Add_to_Apple_Wallet_badge.svg/512px-Add_to_Apple_Wallet_badge.svg.png)

## Overview
This repository contains implementation guidelines and code examples for integrating Apple Wallet functionality into the Cartera TMobilitat application. The integration allows users to add their TMobilitat passes to Apple Wallet with NFC capabilities for contactless validation.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Features](#features)
- [Installation](#installation)
- [Setup](#setup)
- [Implementation](#implementation)
- [Security](#security)
- [Testing](#testing)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites
- Apple Developer Account
- Xcode 14.0 or later
- iOS 13.0+ deployment target
- Pass Type ID certificate
- NFC certificate
- Valid Apple Developer Program membership

## Features
- Add transit passes to Apple Wallet
- NFC validation support
- Real-time pass updates
- Push notifications for pass changes
- Secure encryption for NFC communication
- Server-side pass management

## Installation

### CocoaPods
Add the following to your Podfile:
```ruby
pod 'PassKit'
pod 'CoreNFC'
```

### Swift Package Manager
Add the following to your Package.swift:
```swift
dependencies: [
    .package(url: "your-repository-url", from: "1.0.0")
]
```

## Setup

### 1. Certificate Configuration
1. Log in to [Apple Developer Portal](https://developer.apple.com)
2. Create a Pass Type ID
3. Generate and download the Pass Type ID certificate
4. Generate and download the NFC certificate
5. Add certificates to your Xcode project

### 2. Project Configuration
Add the following entitlements to your project:
```xml
<key>com.apple.developer.pass-type-identifiers</key>
<array>
    <string>$(TeamIdentifierPrefix)*</string>
</array>
<key>com.apple.developer.nfc.readersession.formats</key>
<array>
    <string>NDEF</string>
    <string>TAG</string>
</array>
```

## Implementation

### Pass Generator
Create a pass generator class to handle pass creation:

```swift
import PassKit

class PassGenerator {
    static func createTravelPass(
        serialNumber: String,
        passengerName: String,
        validFrom: Date,
        validUntil: Date
    ) throws -> PKPass {
        // Implementation details in PassGenerator.swift
    }
}
```

### Add to Wallet Button
Implement the Add to Wallet functionality:

```swift
class AddPassViewController: UIViewController {
    private var passKitController: PKAddPassesViewController?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupAddToWalletButton()
    }
    
    // Implementation details in AddPassViewController.swift
}
```

### Server Integration
Set up your server endpoints:
- `/v1/passes/register` - Register device for push notifications
- `/v1/passes/{passTypeIdentifier}/{serialNumber}` - Get latest pass data
- `/v1/devices/{deviceLibraryIdentifier}/registrations/{passTypeIdentifier}` - Handle device registration

## Security

### Best Practices
1. Secure certificate storage
2. HTTPS for all communications
3. Implement rate limiting
4. Validate all requests
5. Encrypt sensitive data
6. Implement proper authentication

### Key Management
- Store private keys securely
- Rotate encryption keys periodically
- Use secure key distribution methods

## Testing

### Test Cases
1. Pass Creation
   - Verify pass generation
   - Validate pass fields
   - Check image assets

2. NFC Functionality
   - Test with physical NFC readers
   - Verify encryption
   - Test different device models

3. Server Integration
   - Test registration flow
   - Verify update mechanism
   - Test push notifications

### Debug Tools
- Apple Wallet Tester
- NFC Reader simulation
- Charles Proxy for network debugging

## Documentation
Detailed documentation is available in the `/docs` directory:
- [Technical Specification](docs/TECHNICAL_SPEC.md)
- [API Documentation](docs/API.md)
- [Security Guidelines](docs/SECURITY.md)

## Contributing
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License
This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## Support
For support, please open an issue in the GitHub repository or contact the development team.

---
Made with frustration by Eliya Maor (SudoSaturn)
