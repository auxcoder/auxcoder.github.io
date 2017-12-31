---
layout: post
title: Mac Finder preferences
author: Auxcoder
categories: Misc
date: 2015-08-10 10:58:48
tags:
- MacOS
comments: false
---


## Set prefered Layout

There are four different layouts that you can choose in the Finder.

* Cover Flow Layout: **Flwv**
* List View: **Nlsv**
* Column View: **clmv**
* Icon View: **icnv**

```sh
defaults write com.apple.Finder FXPreferredViewStyle clmv
```
Where _clmv_ is the layout name.

Restar finder to take effects changes

```sh
killall Finder
```

### Reset layout settings.

```sh
defaults write com.apple.Finder FXPreferredViewStyle icnv
```


### Set item arrangement to none (enables folder dropdowns, 'Name' if you want to remove them)

```sh
defaults write com.apple.finder FXPreferredGroupBy -string "None"
```

### Sort list view by kind in ascending order (Windows style)

```sh
/usr/libexec/PlistBuddy -c "Set :StandardViewSettings:ExtendedListViewSettings:sortColumn kind" ~/Library/Preferences/com.apple.finder.plist
/usr/libexec/PlistBuddy -c "Set :StandardViewSettings:ExtendedListViewSettings:columns:4:ascending true" ~/Library/Preferences/com.apple.finder.plist
/usr/libexec/PlistBuddy -c "Set :StandardViewSettings:ListViewSettings:sortColumn kind" ~/Library/Preferences/com.apple.finder.plist
/usr/libexec/PlistBuddy -c "Set :StandardViewSettings:ListViewSettings:columns:kind:ascending true" ~/Library/Preferences/com.apple.finder.plist
```

### Sort Folders before files hack (Windows style)
```sh
FILE=/System/Library/CoreServices/Finder.app/Contents/Resources/English.lproj/InfoPlist.strings
```

### Backup InfoPlist.strings first if no backup exists
```sh
[ -f $FILE.bak ] || sudo ditto $FILE $FILE.bak
```

### Convert InfoPlist.strings to XML
```sh
sudo plutil -convert xml1 $FILE
```

## Add a space in front of 'Folder' string
```sh
sudo sed -i '' 's/g\>Folder/g\> Folder/' $FILE > /dev/null
```

## Convert InfoPlist.strings back to binary
```sh
sudo plutil -convert binary1 $FILE
```

## Remove all .DS_Store files to reset folder specific view options
```sh
sudo find / -name ".DS_Store" -depth -exec rm -f {} \;
```

## Restart cfprefsd and Finder
```sh
killAll cfprefsd
killAll Finder
```
