---
layout: post
title: Using Fastlane with MS DevOps and Xamarin
date: '2019-08-02 11:00:00'
background: '/img/posts/2019-08-02/header.jpg'
---

A few days ago I started to work on a CI using DevOps with YAML (first time with YAML) and also decided to give Fastlane match a chance since it seemed a nice solution to all those provisioning headaches.
Little did I know, that there was no plugin for it, no example and the documentation wasn't enough... or that would take me that long to figure how to do it.

Now that's done, I have to say that I'm pretty happy with the result, altho probably there's some space for improvements.

Preparing Fastlane

I don't intend to cover all the points on setting up Fastlane itself, on that matter the [documentation](https://docs.fastlane.tools/actions/match/) is pretty good, but I will quickly sum up what I did:

Create a new git repository inside your DevOps to store the certificates and provisioning profiles, I called mine MobileCertificates. Give access to all developers on the team.

Create a shared email account, let's call it *appdevelopers@companyname.com*. Add it to the developer portal and give admin rights, so it can create certificates and provisioning profiles.

Register your app in the Apple Developer Portal and remember the bundle id for later. For this post, I'll consider *com.companyname.projectName*.

On your local computer:
- Install Fastlane: `brew cask install fastlane`
- Go to your project root folder and run: `fastlane init`

Every time I run this, Fastlane "complains" a bit and ask you to configure it manually: 
```
[00:01:30]: Created new folder './fastlane'.
[00:01:30]: No iOS or Android projects were found in directory '/Users/Giusepe/Git/ReactiveUI/ReactiveUI.Samples/xamarin-forms/Cinephile'
[00:01:30]: Make sure to `cd` into the directory containing your iOS or Android app
[00:01:30]: Alternatively, would you like to manually set up a fastlane config in the current directory instead? (y/n)
```
Just hit `y`  and answer the questions.

Go to your project root folder and run: ```fastlane match init```
Select the git repository as a storage and type or paste the path to your repo, something like *https://companyname.visualstudio.com/projectName/_git/MobileCertificates*.

Now there's a folder called `fastlane`, do a `cd fastlane` and edit the `Matchfile`, here is a sample content:

```ruby
type("development")

storage_mode("git")
git_url("https://companyname.visualstudio.com/projectName/_git/MobileCertificates")

app_identifier(["com.companyname.projectName"])
username("appdevelopers@companyname.com") # Your Apple Developer Portal username%
```

Now, before we start creating our certificates I highly recommend that you wipe all the certificates and provisioning profiles and start from scratch. Any app published to the app stores will continue to work, but you will need to regenerate new certificates and provisioning profiles next time you want to update it.

So run 
```bash
fastlane match nuke development
fastlane match nuke distribution
```

Now it's time to create our new certificates:
- `fastlane match appstore` for the app store version
- `fastlane match adhoc` for the beta testing when using App Center
- `fastlane match development`  for... well, development 

We now need to change the configuration on the iOS project to use these new certificates and profiles, so open your solution in Visual Studio, go to the iOS project properties => iOS Bundle Signing

Select the build configuration you use for development (remember to change for both the device and simulator in this case: Debug/iPhoneSimulator and Debug/iPhone)
- Signing Identity: Developer (Automatic)
- Provisioning Profile: match Development com.companyname.projectName

Select the build configuration you use for AdHoc
- Signing Identity: Distribution (Automatic)
- Provisioning Profile: match AdHock com.companyname.projectName

Select the build configuration you use for App Store
- Signing Identity: Distribution (Automatic)
- Provisioning Profile: match AppStore com.companyname.projectName

Make sure you have saved all of them.

Now it's a good moment to try to build and deploy it locally.
If everything is working, commit, push your changes and let's move to the next phase.

Notice that at this point, any other developer just need to install fastlane, run `fastlane match development` and will be ready to work.

Well, remember that I said there's no plugin or task to handle fastlane match inside DevOps? So the solution I found was to write a bash script and call it with a bash task.
Here is how my script (`fastlane.sh`) looks like: 

```bash
#!/bin/bash
if [ "$1" == "devops" ]; then
    export KEYCHAIN_NAME="temporary.keychain"
    export KEYCHAIN_PASSWORD="doesntmatteritwillbetemporaryanyway"

    echo "Setting security"
    security create-keychain -p $KEYCHAIN_PASSWORD $KEYCHAIN_NAME
    security list-keychains -s $KEYCHAIN_NAME
    security unlock-keychain -p $KEYCHAIN_PASSWORD $KEYCHAIN_NAME

    sudo gem install fastlane
    fastlane ci
else
  echo "Can only be run on DevOps."
fi
```

So the first thing I want to mention here is the `if [ "$1" == "devops" ]; then`, I added that after running the script on my computer and almost lost my keychains... so be warned: don't run this on your local computer with the "devops" parameter.

If you try to run fastlane match directly inside DevOps with the default keychain, it will most likely fail and or your build will hang on the signing process because the agent will be asking for a password in some windows somewhere in the cloud and nobody can answer that.
So the solution is to create a temporary keychain and unlock it. 
```bash
echo "Setting security"
security create-keychain -p $KEYCHAIN_PASSWORD $KEYCHAIN_NAME
security list-keychains -s $KEYCHAIN_NAME
security unlock-keychain -p $KEYCHAIN_PASSWORD $KEYCHAIN_NAME
```

With that problem solved, we can finally call fastlane
```bash
    sudo gem install fastlane
    fastlane ci
```

Don't forget to give execution rights to the script `chmod +x fastlane.sh`, commit and push it.

Now, if you just try to run it like this it will fail because we haven't defined the `ci` lane yet, so let's do it.

Let's edit `fastlane/Fastfile` and create a `ci` lane, mine looks something like this:
```ruby
update_fastlane

default_platform(:ios)

platform :ios do 
  desc "Download the certificates and profiles for the CI"
  lane :ci do |options|
    match(type: "appstore", 
          readonly: true,
          force_for_new_devices: false, 
          git_url: "https://test:#{ENV["SYSTEM_ACCESSTOKEN"]}@companyname.visualstudio.com/projectName/_git/MobileCertificates", 
          keychain_name: "temporary", 
          keychain_password: "doesntmatteritwillbetemporaryanyway")
    
    match(type: "adhoc", 
          readonly: true,
          force_for_new_devices: true,
          git_url: "https://test:#{ENV["SYSTEM_ACCESSTOKEN"]}@companyname.visualstudio.com/projectName/_git/MobileCertificates", 
          keychain_name: "temporary", 
          keychain_password: "doesntmatteritwillbetemporaryanyway")
  end
end
```

This lame makes sure we are running the match using the keychain we defined in fastlane.sh and also passes the SYSTEM_ACCESSTOKEN so that the fastlene can access the git repo from the ci machine.

Now it's time to YAML (https://noyaml.com but yeah, better than using the classic UI from DevOps)

Here is how my step for building Xamarin.iOS looks like: 

```yaml    
    steps:
      - task: Bash@3
        displayName: 'Fastlane Match'
        inputs:
          filePath: ci/fastlane.sh
          arguments: "devops"
        env:
          SYSTEM_ACCESSTOKEN: $(System.AccessToken)
          MATCH_PASSWORD: $(match.password)
        enabled: true
      - script: sudo $AGENT_HOMEDIRECTORY/scripts/select-xamarin-sdk.sh 5_18_1
        displayName: 'Select the Xamarin SDK version'
        enabled: true
      - task: NuGetToolInstaller@0
      - task: NuGetCommand@2
        displayName: 'NuGet restore'
        inputs:
          restoreSolution: '**/*.sln'
          vstsFeed: '...'
      - task: XamariniOS@2
        inputs:
          solutionFile: '**/MyAppProject.iOS.csproj'
          configuration: 'Release'
          signingIdentity: 'iPhone Distribution'
          signingProvisioningProfileID: 'match AdHoc com.companyname.projectName'
          packageApp: true
      - task: PublishBuildArtifacts@1
        displayName: 'Publish Artifact: iOS IPA'
        inputs:
          PathtoPublish: '$(outputDirectoryiOS)/MyAppProject.iOS.ipa'
          ArtifactName: 'iOS IPA'
```

The only two differences here from the traditional build without fastlane are:

1) The script task where we set 2 environment variables and call fastlane.sh. 
It's important to mention that while `$(System.AccessToken)` is a DevOps variable and you don't need to set any value before using, `$(match.password)` is a user-defined secret variable and it's where you will put your match encryption key. 

```yaml
     - task: Bash@3
        displayName: 'Fastlane Match'
        inputs:
          filePath: ci/fastlane.sh
          arguments: "devops"
        env:
          SYSTEM_ACCESSTOKEN: $(System.AccessToken)
          MATCH_PASSWORD: $(match.password)
        enabled: true
```



2) The way we set the provisioning profile on the build step:
```yaml
     - task: XamariniOS@2
        inputs:
       ...
          signingIdentity: 'iPhone Distribution'
          signingProvisioningProfileID: 'match AdHoc com.companyname.projectName'
```

I looked into the `*iOS.csproj` and looked at how it was after the fastlane configurations and copied here. 
This snippet is set for an AdHoc build, if I wanted to do an App Store one all I need to change is `signingProvisioningProfileID: 'match AppStore com.companyname.projectName'`

Well, we are all set here... just commit, push and cross fingers :)

I doubt that this will work right away for anyone, it never does... but I hope that with this steps I can save you some of the headaches I have gone thru and also save you some time.

Looking back at all of this, I think that the ideal solution would be to use this knowledge to build a plugin for DevOps and simplify this process even more. 

Feel free to share your experience with me and/or report any issues with this "tutotial". 