
# Assumptions

We build user interfaces in code, and thus also set the text contents from code using NSLocalizedString()

This document is all about Localization. Not about Internationalization.

# Create a Project

Create an Single View Swift project. Add the following to AppDelegate.swift:

```
func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    let message = String.localizedStringWithFormat(NSLocalizedString("Hello, %@! Welcome to %@!",
        tableName: "AppDelegate", comment: "Welcome message when the app starts"), "Stefan", "Mozilla")
    print(message)
    return true
}

```

This is the code that we are going to localize. As you can see the base language in our code is English. Xcode doesn't really know it is english, it simply considers it the Development Language which is English for us.

The first argument to `NSLocalizedString()` is the key of this string in the string table. For the base language the key is the same as the value. This is a good convention but they do not have to but I think it makes things simpler because that means the code is leading and all the other localizations are derived from it.

The tableName allows us to have multipe string files. This makes it easier for the localizers because related strings are close together.

Also note that the string we are going to localize in this case has a format string. Format strings are actually converted in a special way so that it is easy to change for example the order of the string parameters in a localized language. The above string will look like this in the Localizable.strings file:

```
/* Welcome message when the app starts */
"Hello, %@! Welcome to %@!" = "Hello, %1$@! Welcome to %1$@!";
```

If you build and run this project, you should see this in the console. It doesn't matter what locale your device is set to since english is the only locale we have in this app right now.

```
Hello, Stefan! Welcome to Mozilla!
```

Code-wise we are done.



# Add locales to the project

We do not actualy have to manually add locales to the project. What we do is export our supported languages to XLIFF files. Those are then translated. And imported back into the application. Xcode will add locales to the project for those that are not already in there as part of the import process.



# Exporting the localized strings

We are not localizing the strings from within xcode. Instead we are exporting them to .xliff file which can then be translated and imported back into the project.

We are going to swith to the command line for this. Open a terminal window and navigate to your project. Now execute this:

```
cd LocalizationTest
mkdir LocalizedFiles
xcodebuild -exportLocalizations -project LocalizationTest.xcodeproj -localizationPath LocalizedFiles -exportLanguage fr
xcodebuild -exportLocalizations -project LocalizationTest.xcodeproj -localizationPath LocalizedFiles -exportLanguage nl-NL
```

Note that we do not export the base development language, English, since we maintain that in code. If more convenient then we could actually also export/import it and let people manage it completely on localize.mozilla.org

> I'm showing how to do this from the command line because the export, plus upload to localize.mozilla.org, is something we will want to automate.

We now have a LocalizedFiles directory containing xliff files for all our languages. But since these are new, they all contain English and need to be translated.



# Translating 

You can use any translation tool that deal with XLIFF files. You can even just use a text editor. In case of mozilla, we will upload the .xliff files to http://localize.mozilla.org, let our translaters work on the strings and then export them back.

> Downloading the files from localize.mozilla.org can be automated using a simple shell script that loops over our supported locales and then uses `curl` to fetch them from our translation server.

For now, just edit the files manually and replace the strings.

For French use the following translation unit:

```
<trans-unit id="Hello, %@! Welcome to %@!">
  <source>Hello, %1$@! Welcome to %2$@!</source>
  <target>Bonjour, %1$@! Bienvenue à %2$@!</target>
  <note>Welcome message when the app starts</note>
</trans-unit>
```

For Dutch use the following translation unit:

```
<trans-unit id="Hello, %@! Welcome to %@!">
  <source>Hello, %1$@! Welcome to %2$@!</source>
  <target>Hallo, %1$@! Welkom bij %2$@!</target>
  <note>Welcome message when the app starts</note>
</trans-unit>
```

> Note that these files are not awesome to edit by hand. This is why we have a tool like https://localize.mozilla.org

You have now succesfully translated the application in two languages. But these translations live in external files. Or if we had used localize.mozilla.org, live in an external system.

So the next step is importing these files into the project.



# Importing locales into the project

Also using the command line:

```
cd LocalizationTest
mkdir LocalizedFiles
xcodebuild -importLocalizations -project LocalizationTest.xcodeproj -localizationPath LocalizedFiles
```

If you go the the Project settings, you should now see two added Locales: French and Dutch.

NOPE NOPE NOPE IGNORE ALL THAT, XCODEBUILD DOES NOT ACTUALLY HAVE AN -IMPORTLOCALIATIONS OPTION

Actually:

In Xcode, navigate to your project settings. From the Editor menu, select 'Import Localizations'. Go to the LocalizedFiles folder where we exported and manually edited our files. Then import the `fr.xliff` file and do the same for the `nl-NL.xliff` file.

> If xcodebuild cannot do this, maybe we can automate this with AppleScript?

In your project settings you should now see Dutch and French appear under Localizations section.




# Running the application with a specific locale

Go to Product, Scheme, Edit Scheme.

Select the Run action on the left and change the Application Language to Dutch.

Now run the app. In the console you should see:

```
Hallo, Stefan! Welkom bij Mozilla!
```



# Updating localizations

So it is really important to realize that as soon as you upload your .xliff files to localize.mozilla.org, that external system becomes the owner of those translations. This means that before you add new strings to the xcode project, you should first make sure that the project is up to date and that you have imported the most recent version that is kept on localize.mozilla.org. After that you export the localizations again and import them into localize.mozilla.org.

> TODO Does localize.mozilla.org also have the concept of a Base Development Language? If that is the case, we can just push changes to the english locale to l.m.o.


