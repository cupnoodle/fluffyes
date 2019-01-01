# Getting started on building your own backend REST API using Sinatra



Ever wondered how a REST API that returns JSON is being created? You asked around and many suggestion came up: Python Flask Django Ruby Rails Sinatra Node js express Swift Kitura etc, which should you choose? 😱 



In this post, we will be using Ruby (the language) and Sinatra (the framework/library to build a web server easily). We choose Ruby as its syntax is similar to Swift and it is widely used as server side language, this post won't cover the whole Ruby syntax but will explain what does each line of ruby code do.



This post will cover :

1. Installing Ruby and Sinatra
2. Writing your first REST API
3. Return JSON for API
4. Deploy your REST API online to Heroku
5. Connect your REST API in iOS app using URLSession



## Installing Ruby and Sinatra

macOS has bundled a system version of Ruby, but it is few versions behind the latest version and heavily restricted. We will proceed to install **Homebrew** (to install and compile many mac command line software), **rbenv** (to manage multiple version of ruby) and then **Ruby**.



Open up Terminal, then run the following command :

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

<br>



After homebrew has finished installing, we can use it to install rbenv and Ruby.

To install rbenv, run the following commands in Terminal : 

```bash
brew install rbenv ruby-build

# Add rbenv to bash profile so that it initalize every time you open a terminal, changing the ruby version from system default to your defined ones

echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bash_profile
source ~/.bash_profile
```

<br>



Then proceed to install Ruby, we will be using ruby version 2.5.3 : 

```bash
# Install Ruby version 2.5.3 and set it as the default version across the system
rbenv install 2.5.3
rbenv global 2.5.3
```

<br>



To verify you have installed ruby correctly, run `ruby -v` in terminal, you should see the version 2.5.3.

```bash
ruby -v
# should show 2.5.3
```

<br>



Now that we have installed ruby, we can proceed to install Sinatra. Run the following command into Terminal :

```swift
gem install sinatra
```



Now we have finished installing Ruby and Sinatra, let's move on to the fun part next! 🙌 



## Writing your first REST API



