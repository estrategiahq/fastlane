<p align="center">
  <b>Fastlane</b><br />
  <a href="https://github.com/KrauseFx/deliver">Deliver</a> &bull; 
  <a href="https://github.com/KrauseFx/snapshot">Snapshot</a> &bull; 
  <a href="https://github.com/KrauseFx/frameit">FrameIt</a> &bull; 
  <a href="https://github.com/KrauseFx/PEM">PEM</a> &bull; 
  <a href="https://github.com/KrauseFx/sigh">Sigh</a>
</p>
-------

<p align="center">
    
</p>

Fastlane - iOS Deployment without the hassle
============

[![Twitter: @KauseFx](https://img.shields.io/badge/contact-@KrauseFx-blue.svg?style=flat)](https://twitter.com/KrauseFx)
[![License](http://img.shields.io/badge/license-MIT-green.svg?style=flat)](https://github.com/KrauseFx/fastlane/blob/master/LICENSE)
[![Gem](https://img.shields.io/gem/v/fastlane.svg?style=flat)](http://rubygems.org/gems/fastlane)
[![Build Status](https://img.shields.io/travis/KrauseFx/fastlane/master.svg?style=flat)](https://travis-ci.org/KrauseFx/fastlane)

## Beta Notes
Please download this repository as zip and run
```
sudo bundle install
sudo rake install
```

Then run through each gem you are using (deliver, snapshot, frameit, sigh, PEM) on GitHub and download the latest version from the **develop** branch.
For each app run the same commands
```
sudo bundle install
sudo rake install
```

**Please read the rest of the README**

Report any feedback directly to me via [Twitter](https://twitter.com/KrauseFx) or email.


Automate the **whole** deployment process of your iOS apps using ```fastlane``` and all its tools:

- [```deliver```](https://github.com/KrauseFx/deliver): Uploads app screenshots, metadata and app updates to iTunes Connect
- [```snapshot```](https://github.com/KrauseFx/snapshot): Creates perfect screenshots of your app in all languages on all device types automatically
- [```frameit```](https://github.com/KrauseFx/frameit): Adds device frames around your screenshots to use on your website
- [```PEM```](https://github.com/KrauseFx/PEM): Creates push certificates for your server
- [```sigh```](https://github.com/KrauseFx/sigh): Creates, maintains and repairs provisioning profiles for you

Follow the developer on Twitter: [@KrauseFx](https://twitter.com/KrauseFx)


-------
<p align="center">
    <a href="#features">Features</a> &bull; 
    <a href="#installation">Installation</a> &bull; 
    <a href="#quick-start">Quick Start</a> &bull; 
    <a href="#customise-the-fastfile">Customise</a> &bull; 
    <a href="#extensions">Extensions</a> &bull; 
    <a href="#jenkins-integration">Jenkins</a> &bull; 
    <a href="#tips">Tips</a> &bull; 
    <a href="#need-help">Need help?</a>
</p>

-------


# Features
- Connect all tools, part of the ```fastlane``` toolchain to work seamlessly together
- Define different ```deployment lanes``` for App Store deployment, beta builds or testing
- Deploy from any computer
- [Jenkins Integration](#jenkins-integration): Show the output directly in the Jenkins test results
- Store data like the ```Bundle Identifier``` or your ```Apple ID``` once and use it across all tools
- Never remember any difficult commands, just ```fastlane```
- Store **everything** in git. Never lookup the used build commands in the ```Jenkins``` configs
- Saves you **hours** of preparing app submission, uploading screenshots and deploying the app for each update.
- Once up and running, you have a fully working **Continuous Deployment** process. Just trigger ```fastlane``` and you're good to go.

# Installation

Install the gem **NOT DURING BETA***

    sudo gem install fastlane

Make sure, you have the latest version of the Xcode command line tools installed:

    xcode-select --install


# Quick Start


The guide will create all the necessary files for you, using the existing app metadata from iTunes Connect.

- ```cd [your_project_folder]```
- ```fastlane init```
- Follow the guide, which will set up ```fastlane``` for you
- Further customise the ```Fastfile``` using the next section

# Customise the ```Fastfile```
Why should you have to remember complicated commands and parameters?

Store your configuration in a text file to easily deploy from any computer.

Open the ```Fastfile``` using a text editor and customise it even further. (Switch to *Ruby* Syntax Highlighting)

### Lanes
You can define multiple ```lanes``` which are different workflows for a release process.

Examples are: ```appstore```, ```beta``` and ```test```.

You define a ```lane``` like this (more details about the commands in the [Actions](#actions) section):
```ruby
lane :appstore do
  xctool                  # run unit tests
  increment_build_number  # increment the build number by one
  snapshot                # create new screenshots
  sigh                    # download/generate the latest provisioning profile
  deliver                 # upload the screenshots, metadata and app to Apple
  frameit                 # Add device frames around the screenshots
  sh "./upload_screenshots_to_s3.sh" # Example

  say "Successfully deployed new version to the App Store!"
end
```

To launch the ```appstore``` lane run
```
fastlane appstore
```

When one command fails, the execution will be aborted.


### Actions
There are some predefined actions you can use. If you have ideas for more, please let me know.

#### [CocoaPods](http://cocoapods.org)
Everyone using [CocoaPods](http://cocoapods.org) will probably want to run a ```pod install``` before running tests and building the app. 
```ruby
install_cocoapods # this will run pod install
```


#### [xctool](https://github.com/facebook/xctool)
You can run any xctool action. This will require having [xctool](https://github.com/facebook/xctool) installed throw [homebrew](http://brew.sh/).
```ruby
xctool "test"
```

#### [snapshot](https://github.com/KrauseFx/snapshot)
```ruby
snapshot
```

To make `snapshot` work without user interaction, follow the [CI-Guide of `snapshot`](https://github.com/KrauseFx/snapshot/tree/develop#run-in-continuous-integration).

#### [sigh](https://github.com/KrauseFx/sigh)
This will generate and download your App Store provisioning profile. ```sigh``` will store the generated profile in the ```./fastlane``` folder.
```ruby
sigh
```

To use the Ad Hoc profile instead
```ruby
sigh :adhoc
```

#### [deliver](https://github.com/KrauseFx/deliver)
```ruby
deliver
```

If you don't want a PDF report, which you have to approve first, append ```:force``` to the command. This is useful when running ```fastlane``` on your Continuous Integration server.
```ruby
deliver :force
```

- ```deliver :beta```: Upload a beta build for Apple TestFlight
- ```deliver :skip_deploy```: To don't submit the app for review (works with both App Store and beta builds)
- ```deliver :force, :skip_deploy```: Combine options using ```,```

#### [frameit](https://github.com/KrauseFx/frameit)
By default, the device color will be black
```ruby
frameit
```

To use white (sorry, silver) device frames
```ruby
frameit :silver
```

#### [increment_build_number](https://developer.apple.com/library/ios/qa/qa1827/_index.html)
This method will increment the **build number**, not the app version. Usually this is just an auto incremented number. You first have to [set up your Xcode project](https://developer.apple.com/library/ios/qa/qa1827/_index.html), if you haven't done it already.

```ruby
increment_build_number
```

To set your own build number use
```ruby
increment_build_number '75'
```

#### Custom Scripts
Run your own commands using
```ruby
sh "./your_bash_script.sh
```

### *before_all* block
This block will get executed before running the requested lane. It supports the same actions as lanes do.

```ruby
before_all do
  install_cocoapods
  xctool "test"
end
```

### *after_all* block
This block will get executed after running the requested lane. It supports the same actions as lanes do.

```ruby
after_all do
  say "Successfully finished deployment!"
  sh "./send_screenshots_to_team.sh" # Example
end
```

# Extensions
Why only use the default actions? Create your own to extend the functionality of `fastlane`.

Just run `fastlane new_action`. Enter the name of the action and edit the generated Ruby file in `fastlane/actions/[action_name].rb`.

From then on, you can just start using your action in your `Fastfile`.

If you think your extension can be used by other developers as well, let me know, and we can bundle it with `fastlane`.

# Jenkins Integration

`fastlane` automatically generates a JUnit report for you. This allows Continuous Integration systems, like `Jenkins`, access the results of your deployment. 

## Installation
The recommended way to install [Jenkins](http://jenkins-ci.org/) is through [homebrew](http://brew.sh/):

```brew update && brew install jenkins```

From now on start ```Jenkins``` by running:
```
jenkins
```

To store the password in the Keychain of your remote machine, I recommend running `sigh` or `deliver` using ssh or remote desktop at least once.

If you're using `Jenkins` as its own user, you might run into problems with `homebrew` and `phantomjs`.

## Deploy Strategy

You should **not** deploy a new App Store update after every commit, since you still have to wait 1-2 weeks for the review. Instead I recommend using Git Tags, or custom triggers to deploy a new update. 

You can set up your own ```Release``` job, which is only triggered manually.

## Plugins

I recommend the following plugins:

- **[HTML Publisher Plugin](https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin):** Can be used to show the generated screenshots right inside Jenkins.
- **[AnsiColor Plugin](https://wiki.jenkins-ci.org/display/JENKINS/AnsiColor+Plugin):** Used to show the coloured output of the fastlane tools. Dont' forget to enable `Color ANSI Console Output` in the `Build Environment` or your project.
- **[Rebuild Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Rebuild+Plugin):** This plugin will save you a lot of time.

## Build Step
Use the following as your build step:
```
fastlane appstore --trace
```
Replace `appstore` with the lane you want to use. I recommend appending the `--trace` to make debugging easier in case something goes wrong. 

## Test Results and Screenshtos

To show the **deployment result** right in `Jenkins`

- *Add post-build action*
- *Publish JUnit test result report*
- *Test report XMLs*: `fastlane/report.xml`

To show the **generated screenhots** right in `Jenkins`

- *Add post-build action*
- *Publish HTML reports*
- *HTML directory to archive*: `path/fastlane/screenshots`
- *Index page*: `screenshots.html`

Save and run. The result should look like this: 

![JenkinsIntegration](assets/JenkinsIntegration.png)

# Tips

## The other tools
Check out other tools in this collection to speed up your deployment process:
- [```deliver```](https://github.com/KrauseFx/deliver): Deploy screenshots, app metadata and app updates to the App Store using just one command
- [```snapshot```](https://github.com/KrauseFx/snapshot): Create hundreds of screenshots of your iPhone app... while doing something else.
- [```frameit```](https://github.com/KrauseFx/frameit): Want a device frame around your screenshot? Do it in an instant!
- [```PEM```](https://github.com/KrauseFx/pem): Tired of manually creating and maintaining your push certification profiles?
- [```sigh```](https://github.com/KrauseFx/sigh): Because you would rather spend your time building stuff than fighting provisioning.


## Editing the configuration files like ```Fastfile```
Change syntax highlighting to *Ruby*.

## Advanced
#### Snapshot
To skip cleaning the project on every build
```ruby
snapshot :noclean
```

#### Run multiple ```lanes```
You can run multiple ```lanes``` (in the given order) using
```
fastlane test inhouse appstore
````
Keep in mind the ```before_all``` and ```after_all``` block will be executed for each of the ```lanes```.

#### Hide the `fastlane` folder
Just rename the folder to `.fastlane` in case you don't want it to be visible in the Finder. 

# Credentials
Every code, related to your username and password can be found here: [password_manager.rb](https://github.com/KrauseFx/fastlane/blob/master/lib/fastlane/password_manager.rb)

## Storing in the Keychain
By default, when entering your Apple credentials, they will be stored in the Keychain from Mac OS X. You can easily delete them, by opening the Keychain app switching to *All Items* and searching for "*deliver*".

## Passing using environment variables
```
DELIVER_USER
DELIVER_PASSWORD
```

## Implement your custom solution
All ```fastlane``` tools are based on Ruby, you can take a look at the source to easily implement your own authentication solution.

# Need help?
- If there is a technical problem with ```fastlane```, submit an issue. Run ```fastlane --trace``` to get the stack trace.
- I'm available for contract work - drop me an email: fastlane@krausefx.com

# License
This project is licensed under the terms of the MIT license. See the LICENSE file.

# Contributing

1. Create an issue to discuss about your idea
2. Fork it (https://github.com/KrauseFx/fastlane/fork)
3. Create your feature branch (`git checkout -b my-new-feature`)
4. Commit your changes (`git commit -am 'Add some feature'`)
5. Push to the branch (`git push origin my-new-feature`)
6. Create a new Pull Request
