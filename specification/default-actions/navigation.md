# Navigation actions
These actions are responsible for performing server driven navigation within Nimbus, i.e. they're responsible for replacing the current server driven
screen with another server driven screen, using the native navigation of the platform at hand.

There are two types of navigation in Nimbus:
- Common: performs a navigation in full screen, replaces the full content of the screen, the previous screen is not visible after the navigation
completes.
- Presentation: shows the new screen as a dialog, the previous screen is still visible underneath. 