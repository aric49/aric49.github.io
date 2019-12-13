---
layout: post
title: Liberate Your (Legally Obtained) Nook Ebooks
subtitle: A Guide for Removing DRM on your own personal digital property
gh-badge: [star, fork, follow]
---
# Liberating Your Ebooks

**Disclaimer:** *Please do not use this guide for nefarious purposes. I am publishing this to help ebook owners such as myself who have legally purchased copies of ebooks for personal use.  Do not use this guide for ebooks that do not belong to you. Authors such as myself work really hard to write books and deserve to be paid for their efforts. Don't be a jerk.*

DRM or Digital Rights Management, and I have had a hate/hate relationship since I first started consuming digital media in my teenage years. As an adult, most of media recently has been in streaming formats in which I do not have an expectation of ownership or property rights. However, as a yong adult, I began consuming the majority of my books in digital format on a Nook e-reader.  At the time, Nook was a good option for me since I had the option to download backup copies of my books for posterity directly from the Barnes and Nobel website. It appears in recent years that Barnes and Nobel removed this functionality in an effort to lock you more directly into the Nook platform. <BeginRant>As a free and opensource software junkie,this simply will not fly with me.   After all, I legally purchased these books and they are my personal digital property. What I choose to do with my personal property is my business and no entity has any say over what I can and cannot do with my personal property.</EndRant>

I recently started looking into how I can pull my legally purchased ebooks from the Barnes and Nobel Nook platform to take control over my property.  I wanted to share my findings on my blog in case anyone else wishes to do the same. Pulling your ebooks out of Nook is a 2 step process:

1. Download the epub files
2. Retrieve the DRM decryption key

Once you have these items in hand, you can use the Free and OpenSource software [Calibre](https://calibre-ebook.com/) with the DeDRM plugin to remove DRM and manage your ebook library.

One of the other options available according to my research is a paid Windows applicaion by `epubor`.  This option was a no-go for me as it requires you to use windows, pay for the license for non-opensource software, and finally (possibly the worst), give them the username and password to your Nook account. **For me, this was absolutely a deal-breaker**.  However, there is a better way. Reading through the [issues](https://github.com/apprenticeharper/DeDRM_tools/issues/814) on the DeDRM GitHub repo, I came across a few tips that worked really well for me. 

## What You Need
1. [Android Studio and Android Studio SDK](https://developer.android.com/studio/) - Installing Android Studio for me also installed the Android SDK. This is required to spin up a virtual android device and gain root-level access to it. 
2. [Calibre](https://calibre-ebook.com/)
3. [DeDRM Plugin](https://github.com/apprenticeharper/DeDRM_tools/releases) - Used version 6.6.3 at the time of writing

## Create A Virtual Android Device
1. Start Android Studio and go into `AVD Manager` (top right corner of the screen), and click `+ Create New Virtual Device`.   Similar acavandar's post on the aforementioned GitHub issue, I opted to create a Nexus 5 device using Android 7.1.1 (Google APIs) as the base image.   My first experience creating this virtual device, I opted for a Google Play image, but had to re-create it as the Google Play image does not give you root access to the virtual device. 

2. Once your VM instance has created, open Chrome and manually download the Nook app APK in the virtual device's Chrome browser from [here](https://apkpure.com/nook-read-ebooks-magazines/bn.ereader/versions) - I downloaded version **5.0.2.38**. It will prompt you to install the app once it has been downloaded. **Note:** You need to download the APK manually since the Google Play store is not available on the Google API base image)

3. Sign into your Barnes and Noble Nook account in the virtual Nook app and download the books you wish to retain. 

4. You should have an Android-SDK directory available in your home directory. My is located under: `~/Android/Sdk/platform-tools` -- In this directory there should be an `abd` executable.

5. Start a root shell by executing the command `./adb root`

6. Access this shell using:  `./adb shell`.  This should drop you into a root terminal on the virtual device.

## Access the Android Device and Pull your Nook Data:

1. It seems like the path to the actual nook data changes between device type and Nook application version. For me the relevant data was under `/data/data/com.nook.app/`. If you can't find it there, you should be able to find a Nook directory somewhere under `/data/data`. 

2. Exit out of the root shell and use `adb` to pull your ebooks. The epub files should be under: `/data/data/com.nook.app/files/B&N Downloads/`. Since adb has some issues with &'s and spaces in the directory name, you can pull all of the `files` directory:   `./adb pull /data/data/com.nook.app/files/`.   This should create a `files` directory in the same directory you executed adb from.  

3.  We will also need to get the SQLite database which contains the hash key for decrypting your ebooks:   `./adb pull /data/data/com.nook.app/databases/cchashdata.db`

4. At this point you can move your `*.epub` files and the `.db` to another location that's easier to work with. 

## Accessing your DRM Key and Importing your eBooks into Calibre
1. Install Calibre and the DeDRM plugin using the instructions provided by DeDRM GitHub Repo

2. Use a sqlite3 client (Download [here](https://sqlite.org/download.html), *side note: my platoform-tools directory contained a sqlite executable. I already had it downloaded, so I opted to use my own sqlite3 installation.*) to access the `cchasdata.db` file and retrieve your DRM Key. Note the Key will be 28 characters long and always end in an equals (=) sign. 

```
sqlite3 cchashdata.db
sqlite> select hash from cc_hash_data;
YOURHASHKEYXXXXXXXXXXXXXXXX=
```

3. Create a plain text file in a text editor, paste in the key and save the file with a .b64 file extension. IE:  `mykey.b64`  -- You may not need the .b64 file extension, but all the research I've done thus far recommend it. 

4. In Calibre, from the top tool bar select `Preferences -> Plugins -> expand File Type plugins`, select DeDRM 6.6.3 and click the button `Customize Plugin`

5. Click the button for Barnes and Noble Ebooks -> Import existing keyfiles.  Browse in the file manager and select the key you just created

6. Click `close` and apply the changes in the plugin window.  From the main calibre screen, you should be able to import your epub files from the directory you saved them to above. If your ebooks are already imported, delete them and re-import them.  DeDRM only removes the DRM on books during the import phase. Once the import completes, you should have access to decrypted, DeDRM'ed versions of your legally obtained ebooks! Congratulations!


## End Notes

That wasn't so hard was it?  And it sure beats being locked into a proprietary ecosystem for accessing books that are legally and rightfully your property. Hope this helps!

