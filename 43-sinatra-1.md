# Getting started on building your own backend REST API using Sinatra



Ever wondered how a REST API that returns JSON is being created? You asked around and many suggestion came up: Python Flask Django Ruby Rails Sinatra Node js express Swift Kitura etc, which should you choose? 😱 



In this post, we will be using Ruby (the language) and Sinatra (the framework/library to build a web app easily). We choose Ruby as its syntax is similar to Swift and it is widely used as server side language, this post won't cover the whole Ruby syntax but will explain what does each line of ruby code do.



This post assume you have experience working with connecting HTTP REST API in iOS app, and also experience using Git and Github.




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

Open your favorite text editor (I recommend [Sublime Text](https://www.sublimetext.com)), create a new text file and save it as **app.rb** inside a folder named **rest**. You can store this folder in any location you like inside your Mac.



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
# execute the app.rb file, this will execute the web app
```

<br>



You can also type in "ruby "(with a space), and drag in the file into terminal like this : 

![dragdrop](https://iosimage.s3.amazonaws.com/2019/43-sinatra/dragdrop.gif)







You would see a message in Terminal,  "Sinatra (v2.0.4) has taken the stage on 4567 for development". This means that the web server is already running (on your computer, on the port 4567).



Open up web browser, and type in **localhost:4567/** into the address bar, you would see the text 'Awesome!', congratulations! You have now built a REST API! 🙌



'localhost' means your local computer (the computer you are using right now), and :4567 means the port number.

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



Save it and run the web app again (`ruby app.rb`), remember to stop the server and run again when you make changes to the file, else the web server won't get updated. 



Now when we visit **localhost:4567/jonyive**, we will see the text output below : 

![jony ive](https://iosimage.s3.amazonaws.com/2019/43-sinatra/jonyive.png)

Aluminium 👌, when we visit the path '/jonyive', the web app will return the text 'Aluminium'. When visiting the path '/timcook', we will get 'Gotta raise em Mac price'.



"But iOS app typically parse JSON response right? What for returning plain text?" , I heard you, in the next section, we will modify the code so that it return a JSON (for your app to parse).



## Return JSON from API

To return a JSON response, we will need to install sinatra-contrib gem (ruby library). Run the following command in terminal :

```bash
gem install sinatra-contrib
```

<br>



Then in the web app file **app.rb** , we import the sinatra-contrib library on top, and change the plain text to json like this :

```ruby
# app.rb
require 'sinatra'
require "sinatra/json"

get '/' do
  # return json {'text': 'Awesome!'}
  json text: 'Awesome!'
end

get '/jonyive' do
  # return json {'text': 'Aluminium'}
  json text: 'Aluminium'
end

get '/timcook' do
  # return json {'text': 'Gotta raise em Mac price'}
  json text: 'Gotta raise em Mac price'
end
```

<br>



Run **app.rb** and navigate to **http://localhost:4567/jonyive**, and you will see a json text like this:  



![Jony Ive JSON](https://iosimage.s3.amazonaws.com/2019/43-sinatra/jonyivejson.png)



Yay, we have managed to return json from the API! 🙌 In the next section, we will use URLSession to consume this API in the iOS app.



If you want to return multiple key/values in the json, you can do it like this :
```ruby
json text: 'value', text2: 'value2', text3: 'value3'
```
<br>



## Connect your REST API in iOS app using URLSession

Now you have managed to spin up a REST API on your computer, let's connect to it in iOS app!

Run the **app.rb** to launch the web server if you haven't already.



I have wrote about [how to use URLSession](https://fluffy.es/nsurlsession-urlsession-tutorial/) and [parsing JSON](https://fluffy.es/parse-json-using-decodable-protocol) if you need a refresher.




We can then connect to the REST API using URLSession like this : 

```swift
import Foundation

// Parse the JSON into Quote struct
struct Quote : Codable {
    let text : String
}

let url = URL(string: "http://localhost:4567/jonyive")!

// send a get request to the REST API
URLSession.shared.dataTask(with: url, completionHandler: { data, response, error in
    // ensure there is no error for this HTTP response
    guard error == nil else {
        print ("error: \(error!)")
        return
    }
    
    // ensure there is data returned from this HTTP response
    guard let data = data else {
        print("No data")
        return
    }
    
    // Parse JSON into Quote struct using JSONDecoder
    guard let quote = try? JSONDecoder().decode(Quote.self, from: data) else {
        print("Error: Couldn't decode data into quote")
        return
    }
    
    // 'text is Aluminium'
    print("text is \(quote.text)")
}).resume()
```

<br>



Build and run the app in **Simulator** (inside your Mac / the same computer), so that the iOS app can communicate to the web server. You should see the output 'text is Aluminium' in the console, you have successfully connect your app to your own REST API! 🙌



As the '**localhost**' URL can only be accessed by the same computer, it won't work if you build and run the app on a real iOS device (as it is not the same device running the web server).



As 'localhost' won't work for app running in real device, we will need to upload and run our web server code on the internet. In the next section, we will use Heroku for deploying the web server code.



## Deploy your REST API online to Heroku

Before putting up the app online, we will need to create a file named '**config.ru**', put it in the same folder as 'app.rb'. The content inside '**config.ru**' file is as follow : 

```ruby
# config.ru
require './app'

run Sinatra::Application
```

<br>

This file (.ru stands for rack up) is used to instruct the [rack web server](http://rack.github.io) to run the web app (app.rb) we wrote earlier. This is needed when we put the web app online.



Next, we will also need to create a file named '**Gemfile**' in the same folder, this will be used to instruct Heroku what gem / library will need to be installed.

```ruby
source 'https://rubygems.org'

gem 'sinatra'
gem 'sinatra-contrib'
```

<br>



Before being able to deploy the web server code to Heroku, we will need to initialize a repository to store the code and push it to Github.



Create a new repository in [Github](https://github.com), name it as you like, I name it as '**rest**'.



Open terminal, navigate to the folder containing **app.rb** (the folder named 'rest') , you can use drag and drop like this : 

![navigate folder](https://iosimage.s3.amazonaws.com/2019/43-sinatra/navigateFolder.gif)



Before initializing the git repository, run the command below to generate Gemfile.lock. Heroku will need this later on

```bash
bundle install
```

<br>



Then initialize a git repository on the folder (in you Mac)  and push it to Github like this :

```bash
git init
git add -A
git commit -m "first commit"
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/REPO_NAME.git
git push -u origin master
```

<br>

Remember to change **YOUR_GITHUB_USERNAME** to your github username and **REPO_NAME** to the github repository name.



Now that we have pushed to code to Github repository, let's head over to [Heroku](https://heroku.com). You can [create a free account on Heroku](https://signup.heroku.com) if you haven't had one yet. After creating the account, sign into Heroku and you will ve redirected to the dashboard.



Let's create a new app by clicking 'New' -> 'App' on the top right corner.



![Create New App](https://iosimage.s3.amazonaws.com/2019/43-sinatra/createNewApp.png)



Type in a name and click 'Create app'

![app name](https://iosimage.s3.amazonaws.com/2019/43-sinatra/firstrestapi.png)



Then connect the app to the Github repository you created earlier : 

![connect](https://iosimage.s3.amazonaws.com/2019/43-sinatra/connect.png)



After connecting Github repository, scroll down and you will see a 'Manual Deploy' section, click the 'Deploy Branch' button to put your REST API online! 🚀

![deploy branch](https://iosimage.s3.amazonaws.com/2019/43-sinatra/deployBranch.png)



It will take few seconds to process after you press the button, once it is deployed, Heroku will show a "View" button. Clicking it will bring you to your online web app. Congratulations, you have managed to put your own REST API online! 🎉



![heroku](https://iosimage.s3.amazonaws.com/2019/43-sinatra/heroku.png)



Now you can replace the url in your iOS app from 'localhost:4567/jonyive' to 'https://your-app-name-on-heroku.herokuapp.com' , and you can build and run the app on any iOS device! (You can even submit it to the App Store if you want).





