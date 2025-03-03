# Cartera TMobilitat - Design Guide

## Table of Contents
1. [Overview](#overview)
2. [Pass Layout Structure](#pass-layout-structure)
3. [Pass Lifecycle](#pass-lifecycle)
4. [Custom Design Guidelines](#custom-design-guidelines)
5. [Color Scheme](#color-scheme)
6. [Field Types and Specifications](#field-types-and-specifications)
7. [Responsive Design](#responsive-design)
8. [Implementation Examples](#implementation-examples)
9. [Design Best Practices](#design-best-practices)
10. [Testing Your Design](#testing-your-design)
11. [Troubleshooting](#troubleshooting)

## Overview

This document provides comprehensive guidelines and specifications for designing TMobilitat passes for Apple Wallet. It covers visual design elements, layout structure, branding requirements, and technical specifications for creating visually appealing and functional passes.

## Pass Layout Structure

### Basic Layout
```
+--------------------------------------------------+
|                     Header                         |
|  +--------------------------------------------+  |
|  |                  logo.png                   |  |
|  +--------------------------------------------+  |
|                                                  |
|  Primary Fields                                  |
|  +--------------------------------------------+ |
|  | TÍTOL                                       | |
|  | T-usual                                     | |
|  +--------------------------------------------+ |
|                                                  |
|  Secondary Fields                                |
|  +----------------------+ +--------------------+ |
|  | VÀLID DES DE        | | VÀLID FINS         | |
|  | 15 de març, 2024    | | 15 d'abril, 2024   | |
|  +----------------------+ +--------------------+ |
|                                                  |
|  Auxiliary Fields                                |
|  +----------------------+ +--------------------+ |
|  | TIPUS                | | ZONES              | |
|  | Mensual             | | Totes les zones    | |
|  +----------------------+ +--------------------+ |
|                                                  |
|  Footer                                         |
|  +--------------------------------------------+ |
|  |           Termes i condicions              | |
|  +--------------------------------------------+ |
+--------------------------------------------------+
```

## Pass Lifecycle

### Lifecycle Diagram
```
+---------------+     +--------------+     +---------------+
|               |     |              |     |               |
| Pass Creation |---->| Pass Signing |---->| Add to Wallet|
|               |     |              |     |               |
+---------------+     +--------------+     +---------------+
        ^                                          |
        |                                          v
+---------------+     +--------------+     +---------------+
|               |     |              |     |               |
|  Pass Expired |<----| Validation   |<----| Active Pass   |
|               |     |              |     |               |
+---------------+     +--------------+     +---------------+
                                                |
                            +-------------------)
                            |                   |
                     +---------------+          |
                     |               |          |
                     | Pass Update   |----------+
                     |               |
                     +---------------+
```

## Custom Design Guidelines

### Pass Content Elements
Your pass can include:
- TMobilitat logo
- Pass holder's name
- Valid dates
- Pass type (T-usual, T-casual, etc.)
- Custom colors
- Special messages

### Basic Pass Design Example

```json
{
    "formatVersion": 1,
    "passTypeIdentifier": "pass.tmobilitat.cartera",
    "serialNumber": "123456",
    "teamIdentifier": "YOUR_TEAM_ID",
    "organizationName": "TMobilitat",
    "description": "Títol de Transport",
    
    // Colors (TMobilitat brand colors)
    "backgroundColor": "rgb(0, 155, 216)",
    "foregroundColor": "rgb(255, 255, 255)",
    "labelColor": "rgb(255, 255, 255)",
    
    // The main information shown on the pass
    "generic": {
        "primaryFields": [
            {
                "key": "title",
                "label": "TÍTOL",
                "value": "T-usual"
            }
        ],
        "secondaryFields": [
            {
                "key": "validFrom",
                "label": "VÀLID DES DE",
                "value": "15 de març, 2024"
            },
            {
                "key": "validUntil",
                "label": "VÀLID FINS",
                "value": "15 d'abril, 2024"
            }
        ],
        "auxiliaryFields": [
            {
                "key": "zones",
                "label": "ZONES",
                "value": "Totes les zones"
            }
        ]
    }
}
```

## Color Scheme

```
Background Colors:
+------------------+-------------------------+
| Light Mode       | Dark Mode              |
+------------------+-------------------------+
| Primary:   #009BD8| Primary:   #007AAB     |
| Secondary: #F8F8F8| Secondary: #1C1C1E     |
| Accent:    #FF6B00| Accent:    #FF8533     |
+------------------+-------------------------+

Text Colors:
+------------------+-------------------------+
| Light Mode       | Dark Mode              |
+------------------+-------------------------+
| Primary:   #000000| Primary:   #FFFFFF     |
| Secondary: #666666| Secondary: #EBEBF5     |
| Label:     #009BD8| Label:     #00B0F5     |
+------------------+-------------------------+
```

## Field Types and Specifications

### Images
```
logo.png:    160x50 points  (320x100 pixels @2x)
icon.png:     29x29 points  (58x58 pixels @2x)
```

### Text Fields
```
Primary:     Maximum 3 fields
             Font size: 17pt
             Weight: Medium

Secondary:   Maximum 4 fields
             Font size: 15pt
             Weight: Regular

Auxiliary:   Maximum 4 fields
             Font size: 13pt
             Weight: Regular

Header:      Maximum 1 line
             Font size: 13pt
             Weight: Regular

Footer:      Maximum 1 line
             Font size: 12pt
             Weight: Regular
```

### Layout Grid
```
Margins:
+-----------------+
| Top:     16pt   |
| Bottom:  16pt   |
| Left:    16pt   |
| Right:   16pt   |
+-----------------+

Spacing:
+----------------------+
| Between fields: 10pt  |
| Between groups: 20pt  |
+----------------------+
```

## Responsive Design

### Device Adaptations
```
+--------------------------------------------------+
|                  iPhone SE                        |
|  [Disseny compacte amb marges reduïts]           |
+--------------------------------------------------+

+--------------------------------------------------+
|                  iPhone 14                        |
|  [Disseny estàndard amb marges complets]         |
+--------------------------------------------------+

+--------------------------------------------------+
|                 iPhone 14 Pro Max                 |
|  [Disseny ampliat amb camps addicionals visibles]|
+--------------------------------------------------+
```

### Field Alignment Examples
```
Left Alignment:
+--------------------------------------------+
| TÍTOL                                       |
| T-usual                                     |
+--------------------------------------------+

Center Alignment:
+--------------------------------------------+
|               VÀLID FINS                   |
|              15 d'abril, 2024             |
+--------------------------------------------+

PKTextAlignment Options:
- PKTextAlignmentLeft
- PKTextAlignmentCenter
- PKTextAlignmentRight
- PKTextAlignmentNatural
```

## Implementation Examples

### Swift Implementation

```swift
import PassKit

class TMobilitatPassDesigner {
    func createPassDesign(for passenger: String) -> [String: Any] {
        return [
            // Basic pass information
            "formatVersion": 1,
            "passTypeIdentifier": "pass.tmobilitat.cartera",
            "serialNumber": UUID().uuidString,
            "teamIdentifier": "YOUR_TEAM_ID",
            
            // Visual design
            "backgroundColor": "rgb(0, 155, 216)",
            "foregroundColor": "rgb(255, 255, 255)",
            "labelColor": "rgb(255, 255, 255)",
            
            // Pass information
            "organizationName": "TMobilitat",
            "description": "Títol de Transport",
            
            // Pass content
            "generic": [
                "primaryFields": [
                    [
                        "key": "title",
                        "label": "TÍTOL",
                        "value": "T-usual"
                    ]
                ],
                "secondaryFields": [
                    [
                        "key": "validFrom",
                        "label": "VÀLID DES DE",
                        "value": "15 de març, 2024"
                    ],
                    [
                        "key": "validUntil",
                        "label": "VÀLID FINS",
                        "value": "15 d'abril, 2024"
                    ]
                ],
                "auxiliaryFields": [
                    [
                        "key": "zones",
                        "label": "ZONES",
                        "value": "Totes les zones"
                    ]
                ]
            ]
        ]
    }
}
```

### Adding Images to Your Pass

1. Create these image files:
   - `logo.png` (160x50 points)
   - `logo@2x.png` (320x100 pixels)
   - `icon.png` (29x29 points)
   - `icon@2x.png` (58x58 pixels)

2. Add them to your pass package:
```
TMobilitatPass.pass/
├── logo.png
├── logo@2x.png
├── icon.png
└── icon@2x.png
```

## Design Best Practices

### Tips for Effective Pass Design

1. **Keep it Simple**
   - Don't overcrowd the pass
   - Use clear, readable fonts
   - Leave appropriate white space

2. **Use Consistent Colors**
   - Stick to TMobilitat brand colors
   - Ensure text is readable against backgrounds
   - Test on both light and dark modes

3. **Information Hierarchy**
   - Put the most important info in primary fields
   - Use secondary fields for supporting info
   - Keep auxiliary fields for extra details

4. **Localization**
   - Use Catalan as the primary language
   - Consider multilingual support for tourists
   - Ensure date formats follow local conventions

5. **Accessibility**
   - Maintain sufficient contrast ratios
   - Use appropriate font sizes
   - Test with VoiceOver enabled

## Testing Your Design

1. Build and run your app
2. Add the pass to Wallet
3. Check how it looks:
   - In light mode
   - In dark mode
   - When notifications appear
   - On different iPhone models (SE, standard, Pro Max)
   - With different text lengths

4. Test the pass in real-world scenarios:
   - Opening from lock screen
   - NFC validation
   - Receiving updates

## Troubleshooting

### Common Issues and Solutions

#### "My colors look different in Wallet"
- Ensure you're using RGB values (0-255)
- Test on a real device, not just the simulator
- Check color profiles in your image assets

#### "My logo appears blurry"
- Verify you're using the correct image sizes
- Include both regular (@1x) and retina (@2x) versions
- Use PNG format for best results

#### "Text is truncated or cut off"
- Use shorter text where possible
- Test with maximum expected text length
- Test on smaller iPhone screens
- Consider using multiline text for longer content

#### "Pass doesn't update correctly"
- Verify your webServiceURL is correctly configured
- Check server logs for update request failures
- Test push notification configuration 