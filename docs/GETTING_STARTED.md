# Cartera TMobilitat - Getting Started

## Overview

This guide provides step-by-step instructions for getting started with the Cartera TMobilitat Apple Wallet integration. Follow these steps to quickly set up and implement the TMobilitat pass functionality in your iOS application.

## What We're Building

We're going to create an app that lets users:
1. Add their TMobilitat pass to Apple Wallet
2. Use their phone to tap and validate their pass
3. Get updates about their pass status

## Prerequisites

Before we start, you'll need:
- A Mac computer
- Xcode (free from the Mac App Store)
- An Apple Developer account (you can start with a free one for testing)

## Step-by-Step Guide

### 1. Setting Up Your Development Environment

First, let's get your computer ready:

1. **Install Xcode**
   - Open the Mac App Store
   - Search for "Xcode"
   - Click "Get" or "Install"
   - Wait for the download to complete

2. **Sign up for an Apple Developer Account**
   - Go to [developer.apple.com](https://developer.apple.com)
   - Click "Account" and sign in with your Apple ID
   - Fill in your details

### 2. Creating Your First Project

Let's create your app:

1. Open Xcode
2. Click "Create a new Xcode project"
3. Choose "App" under iOS
4. Fill in your details:
   ```
   Product Name: Cartera TMobilitat
   Organization Identifier: com.yourname.tmobilitat
   Interface: SwiftUI
   Language: Swift
   ```

### 3. Adding Apple Wallet Support

Now, let's add the necessary code that lets us work with Apple Wallet.

Add this to the top of your main Swift file:
```swift
import PassKit // This lets us use Apple Wallet features
```

Then add this simple button to start:
```swift
struct AddToWalletButton: View {
    var body: some View {
        Button(action: {
            addPassToWallet()
        }) {
            Text("Add to Apple Wallet")
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(10)
        }
    }
    
    func addPassToWallet() {
        // We'll add more code here soon
        print("Button tapped!")
    }
}
```

### 4. Creating Your First Pass

Here's a simple example of creating a pass:

```swift
func createBasicPass() {
    // This is where we'll create the pass
    let pass = PKPass(
        data: passFileData, // We'll create this next
        error: nil
    )
    
    // Show the "Add to Wallet" screen
    let passController = PKAddPassesViewController(pass: pass)
    // Show it to the user
}
```

### 5. Testing Your App

Let's test what we've built:
1. Connect your iPhone to your Mac
2. In Xcode, select your iPhone from the device list
3. Click the "Play" button (or press Cmd + R)
4. The app will start on your phone

## Common Questions

### "Help! I got an error!"
Don't worry! Most common errors are easy to fix. Here are the most common ones:

1. **"Could not find developer disk image"**
   - Solution: Update Xcode or your iOS version

2. **"Signing certificate not found"**
   - Solution: In Xcode, go to Preferences → Accounts and sign in with your Apple ID

### "What's Next?"

Once you've got this working, you can:
1. Customize your pass design
2. Add user information
3. Set up NFC validation

## Need Help?

If you get stuck:
1. Check the error message - Xcode usually tells you what's wrong
2. Search for the error message online
3. Ask questions in our GitHub issues
4. Join our developer support channel

## Quick Tips for Success

1. **Take it slow** - Rome wasn't built in a day, and neither are apps
2. **Test often** - Build and run your app frequently to catch problems early
3. **Keep it simple** - Start with the basics, then add features
4. **Save your work** - Use git (version control) to save your progress

## Resources for Learning More

- [Apple's PassKit Tutorial](https://developer.apple.com/documentation/passkit/wallet)
- [SwiftUI Basics](https://developer.apple.com/tutorials/swiftui)
- [Our Video Tutorials](https://youtube.com/your-channel)
- [Sample Code Repository](https://github.com/your-repo)

## Next Steps

Ready to add more features? Check out our other guides:
- [Implementation Guide](docs/IMPLEMENTATION_GUIDE.md)
- [Design Guide](docs/DESIGN_GUIDE.md)
- [Technical Specification](docs/TECHNICAL_SPEC.md) 