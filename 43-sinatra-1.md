# Getting started on building your own backend REST API using Sinatra



Ever wondered how a REST API that returns JSON is being created? You asked around and many suggestion came up: Python Flask Django Ruby Rails Sinatra Node js express Swift Kitura etc, which should you choose? 😱 



In this post, we will be using Ruby (the language) and Sinatra (the framework/library to build a web server easily). We choose Ruby as its syntax is similar to Swift and it is widely used as server side language, this post won't cover the whole Ruby syntax but will explain what does each line of ruby code do.



This post will cover :

1. Installing Ruby and Sinatra

2. Writing your first REST API

3. Return JSON from API

4. Connect your REST API in iOS app using URLSession

5. Deploy your REST API online to Heroku




## Installing Ruby and Sinatra

macOS has bundled a system version of Ruby, but it is few versions behind the latest version and heavily restricted. We will proceed to install **Homebrew** (to install and compile many mac command line software), **rbenv** (to manage multiple version of ruby) and then **Ruby**.



![Terminal](https://iosimage.s3.amazonaws.com/2019/43-sinatra/terminal.png)



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

Open your favorite text editor (I recommend [Sublime Text](https://www.sublimetext.com)), create a new text file and save it as **app.rb** in any location you like inside your Mac.



In the text file, write this : 

```ruby
# app.rb
require 'sinatra'

get '/' do
  'Awesome!'
end
```

<br>



`#app.rb` is a comment, it is similar to Swift comment eg:  `// ViewController.swift`.

`require 'sinatra'` means importing the Sinatra library, it is similar to Swift import eg: `import UIKit`



`get '/' do ... end` is calling a function named **get()** with parameter **'/'** . In Swift, this would look like :

```swift
// return the string 'Awesome' when a GET request is sent to the URL '/'
func get(url: String) -> String{
  return 'Awesome!'
}
```

<br>



Open Terminal, change directory to the folder containing the **app.rb** file, then run

```bash
# cd /path/to/the/folder
ruby app.rb
# execute the app.rb file, this will execute the web server
```

<br>



You can also type in "ruby "(with a space), and drag in the file into terminal like this : 

![dragdrop](https://iosimage.s3.amazonaws.com/2019/43-sinatra/dragdrop.gif)







You would see a message in Terminal,  "Sinatra (v2.0.4) has taken the stage on 4567 for development". This mean that the web server is already running (on your computer, on the port 4567).



Open up web browser, and type in **localhost:4567/** into the address bar, you would see the text 'Awesome!', congratulations! You have now built a REST API! 🙌



![localhost:4567](https://iosimage.s3.amazonaws.com/2019/43-sinatra/localhost4567.png)



To stop the web server, press Control + C in the Terminal. 



We can then proceed to add more function in the **app.rb** to handle more path, eg: 

```ruby
# app.rb
require 'sinatra'

get '/' do
  'Awesome!'
end

get '/jonyive' do
  'Aluminium'
end

get '/timcook' do
  'Gotta raise em Mac price'
end
```

<br>



Save it and run the web server again (`ruby app.rb`), remember to stop the server and run again when you make changes to the file, else the web server won't get updated. 



Now when we visit **localhost:4567/jonyive**, we will see the text output below : 

![jony ive](https://iosimage.s3.amazonaws.com/2019/43-sinatra/jonyive.png)

Aluminium 👌, when we visit the path '/jonyive', the web server will return the text 'Aluminium'. When visiting the path '/timcook', we will get 'Gotta raise em Mac price'.



"But iOS app typically parse JSON response right? What for returning plain text?" , I heard you, in the next section, we will modify the code so that it return a JSON (for your app to parse).



## Return JSON from API



