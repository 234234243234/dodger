# Brawl Stars Auto Dodge - Frequently Asked Questions

## General Questions

### What exactly does this app do?
The app uses computer vision technology to detect projectiles in Brawl Stars and automatically executes dodge movements to help you avoid taking damage during gameplay.

### Is this allowed by Supercell?
This app is not affiliated with or endorsed by Supercell. Third-party applications that interact with games may violate the Terms of Service. Use at your own discretion and risk.

### Will this get my account banned?
While we designed the app to be as unobtrusive as possible, using third-party tools always carries some risk. The app only uses the accessibility service to perform actions a human could do, but Supercell's policies on such tools may change.

### Does this work for all brawlers?
Yes, the app is designed to detect projectiles from all brawlers in the game. The detection algorithm looks for visual patterns and movement that are common to all projectiles regardless of which brawler fired them.

## Technical Questions

### Why does the app need so many permissions?
- **Screen Capture Permission**: Needed to see the game screen and detect projectiles
- **Overlay Permission**: Required for the floating control panel
- **Accessibility Service**: Needed to perform dodge movements automatically

### Does this app work on all Android devices?
The app should work on all Android devices running Android 7.0 (Nougat) or higher. However, performance may vary based on your device's processing power. Mid to high-end devices will provide the best experience.

### Does this drain my battery quickly?
Yes, the app requires continuous screen capture and real-time image processing, which can be resource-intensive. We recommend playing while your device is charging for longer gaming sessions.

### Will this work with my screen protector/case?
Yes, since the app captures the screen directly from the system (not using a camera), physical screen protectors or cases will not affect functionality.

## Performance Questions

### The app isn't detecting projectiles properly. What can I do?
Try the following:
1. Make sure you've granted all permissions
2. Restart the app
3. Lower your game's graphics settings 
4. Close other background apps to free up resources
5. Make sure your screen brightness is at a reasonable level

### The dodge movements are too slow. Can I adjust them?
The current version uses fixed dodge parameters. Future updates may include customizable dodge sensitivity and distance settings.

### Can I use other controls while Auto Dodge is active?
Yes, you can continue to move, attack, and use Super abilities as normal. The auto dodge will briefly take control when needed to avoid projectiles, then return control to you.

### The app makes my game lag. How can I fix this?
The image processing requires significant resources. Try:
1. Lowering game graphics settings
2. Closing background apps
3. Restarting your device
4. If possible, use a more powerful device

## Usage Questions

### Can I turn off Auto Dodge temporarily?
Yes, you can toggle the switch on the floating control panel to disable and enable the feature at any time without closing the app.

### Will Auto Dodge work in all game modes?
Yes, the core functionality works in all game modes. However, it may be more effective in some modes than others depending on the pace of gameplay and number of projectiles on screen.

### Does this work with other games?
No, this app is specifically designed for Brawl Stars. The projectile detection algorithm is calibrated for this game's visual style and mechanics.

### Can I customize what gets dodged?
The current version attempts to dodge all detected projectiles. Future updates may include settings to prioritize certain types of threats or to adjust dodge sensitivity.

## Troubleshooting

### The app crashes when starting a game
Try:
1. Restart your device
2. Reinstall the app
3. Make sure you have enough free storage on your device
4. Close other resource-intensive apps

### The floating controls disappeared. What happened?
The overlay might have been accidentally closed or failed. Exit Brawl Stars, restart the Auto Dodge app, and enable the service again.

### The dodge movements are erratic or unpredictable
This can happen if the app detects multiple threats simultaneously. Try:
1. Playing in less chaotic game modes initially
2. Restart the app
3. Switch to a different network if playing online (latency can affect timing)

### I granted all permissions but the Start button is still disabled
Try:
1. Restart the app
2. Make sure you actually enabled the accessibility service (not just viewed the settings)
3. In some cases, you may need to restart your device after granting permissions