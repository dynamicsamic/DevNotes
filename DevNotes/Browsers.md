# Chrome
#### Duplicate current tab
```bash
ctr+l (copy current url)
then 
alt+enter (open a new tab with copied url)
# Not quite duplicate, it just opens a new (last) tab with copied url.
```
---
#### Block `sign-in` popups
```md
1. Open Settings
2. Click on 'Privacy and security' on the sidebar.
3. Select the 'Site settings' option.
4. Expand 'Additional content settings' option.
5. Select 'Third-party sign-in' option.
6. Select the radio 'Block sign-in prompts from identity services'
```
---
#### Create a separate instance of Chrome browser (Windows)
```md
1. Press `win` button.
2. Find Chrome app icon.
3. Click on it with right button.
4. Select `Open file location`.
5. You will see either the executable file or another shortcut.
	a. If you found a shortcut, repeat steps 3-4
6. Right click on the executable.
7. Choose `Show more options`.
8. Choose `Create a shortcut`.
9. It will ask to create a shortcut on your desktop.
10. Agree and navigate to Desktop.
11. Right click on the newly created shortcut.
12. Choose properties.
13. Navigate to the `target` attribute.
14. There you will see the path to executable.
15. You will need a separate directory to store Chrome application files. Create one in appropriate location.
16. Add --user-data-dir="C:\Path\To\Directory after the path.
17. Save the shortcut and run it.
18. Give a meaningful name to your chrome instance.
```
---
# Firefox
>[!info]- Change how new tab behave on open
>- Open new tab
>- In the address bar type `about:config`. A configuration list will be loaded.
>- In the search bar of the configuration list type `curr` to find setting for current tab.
>- Set the `browser.tabs.insertAfterCurrent` to `true`
