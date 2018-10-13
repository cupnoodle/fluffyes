# Upload image to a server using NSURLSessionUploadTask

This post assume you have some experience working with [URLSession](https://fluffy.es/nsurlsession-urlsession-tutorial/) and some brief knowledge on how HTTP works.



You might have come across scenarios where your app need to send some image data to a server API endpoint (eg: user updates profile picture). A quick search on Google / Stack Overflow will get you many answers recommending you to use Alamofire / AFNetworking, but is it really necessary to install a library just for handling image upload? What is happening under the hood of Alamofire / AFNetworking library?



In this post, we will use Apple's own URLSession and its [uploadTask](https://developer.apple.com/documentation/foundation/urlsessionuploadtask) to upload the image to server. We will use [Catbox's free API](https://catbox.moe/tools.php) to upload the image in this post.



The Catbox API endpoint is located at **https://catbox.moe/user/api.php**



According to their [API documentation](https://catbox.moe/tools.php), to upload a file to their server, we will need to send the parameters below : 

**File Uploads**

```
reqtype="fileupload" fileToUpload=(file data here)
```



Before we start to work on the image upload code, let's take a look at how the HTTP request looks like 



