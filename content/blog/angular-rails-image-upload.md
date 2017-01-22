+++
comments = true
date = "2014-07-13T09:03:58+08:00"
showpagemeta = true
categories = ["angularjs", "rails", "image upload"]
slug = ""
draft = false
title = "Image upload tutorial in AngularJS with Rails"
showcomments = true
tags = []

+++

The hardest part of Marathon app is almost done. I managed to connect the view with the model (finally!) and the project assignment CRUD interface is almost finished. I am thankful to be given an opportunity to work on the front-end of a non-trivial app, it really exposes me to the intricacies of AngularJS framework and the complexity of CSS. Even though what really interests me is back-end development where scalability is of paramount importance and where interesting algorithmic problems can be solved, front-end development still continues to delight the creator within me. We humans are all creators at heart. By playing around with front-end technologies, we put ourselves through an incremental process where the product takes shape pixel by pixel right in front of our eyes; like a sculptor freeing his angels from stone using his chisel and hammer, we free our ideas from fantasy by realizing it with modern web technologies.

But I have yet to blog about my learning process.

I used to take blogging for granted. I wrote sappy poems and the cruelties of my schooling life on tumblr last time. It worked to relieve my adolescent angst, but there was no educational value in that pursuit. This blog is different though; it is a collection of software engineering gems, a precious memory bank for the future me to siphon from. This idea never really hit home until one day I was explaining REST to my friend by showing him my blog post. I often refer to my post on ActiveRecord when I’m doing database migrations on Rails too.

Blogging is awesome and today I’m going to explain to my future self how to create an image upload function using Rails and Angular.

<hr>

### Image Upload Tutorial

For image storage, we are going to use a remote file system like Amazon S3 instead of a database. Why? Because storing on a file system is cheaper and accessing images in a file system is much more straightforward than accessing them in a database. However, the database is still important for storing file paths or URLs to the remotely-stored images.

Amazon S3 is an obvious pick. Because it is really simple to set up and there’s a lot of documentation for it out there.

Amazon has a good tutorial on a no-frills method to upload images via the input forms [here](http://aws.amazon.com/articles/1434).

```html
<html>
  <head>
    <title>S3 POST Form</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
  </head>

  <body>
    <form action="https://s3-bucket.s3.amazonaws.com/" method="post" enctype="multipart/form-data">
      <input type="hidden" name="key" value="uploads/${filename}">
      <input type="hidden" name="AWSAccessKeyId" value="YOUR_AWS_ACCESS_KEY">
      <input type="hidden" name="acl" value="private">
      <input type="hidden" name="success_action_redirect" value="http://localhost/">
      <input type="hidden" name="policy" value="YOUR_POLICY_DOCUMENT_BASE64_ENCODED">
      <input type="hidden" name="signature" value="YOUR_CALCULATED_SIGNATURE">
      <input type="hidden" name="Content-Type" value="image/jpeg">
      <!-- Include any additional input fields here -->

      File to upload to S3:
      <input name="file" type="file">
      <br>
      <input type="submit" value="Upload File to S3">
    </form>
  </body>
</html>
```

You can just dump your AWS access key and bucket name on those input form parameters, but you have to encode your policy document (expressed in JSON) and generate a signature by signing your encoded policy document with secret key. This is where server-side languages come in; the encoding and message digest generation can be done using Ruby or Java.

Since I’m a Ruby user, let’s have a look at the code for encoding and generation of signature in Ruby:

```ruby
require 'base64'
require 'openssl'
require 'digest/sha1'

policy = Base64.encode64(policy_document).gsub("\n","")

signature = Base64.encode64(
	OpenSSL::HMAC.digest(
        OpenSSL::Digest::Digest.new('sha1'),
        aws_secret_key, policy)
    ).gsub("\n","")
```

You can just print out the values of `policy` and `signature` to console, copy paste them to the input forms and you’re done. But what if you switched AWS accounts and hence you have to use a different key? The whole process of encoding policy documents and generating signatures gets tedious and it would be better to stick the code into a server, let it run permanently and encode/generate the necessary tokens and send them as a JSON response back to the client.

Hence the client-side code would have placeholders instead of the actual values in the above input form template. Whenever the user uploads a file, it will send a GET request to the server, receive the necessary authentication tokens as a response, pass them to the form, and finally send a POST request to Amazon S3.

This is the basic idea behind the Angular directive (ng-s3upload) that I’m using for my image upload function. You can check it out here. As seen from the source code, it is a little different in that the input form is created programmatically instead of being used as a template.

Having explained the concepts, it is pretty straightforward as seen from the guidelines.

Firstly, the JSON response is generated by this code here. It is used in an Rails/Sinatra app, depending on your framework of choice. Notice that the last two methods used are similar to the ones in Amazon’s documentation.

```ruby
def s3_access_token
  render json: {
    policy:    s3_upload_policy,
    signature: s3_upload_signature,
    key:       GLOBAL[:aws_key]
  }
end

protected

  def s3_upload_policy
    @policy ||= create_s3_upload_policy
  end

  def create_s3_upload_policy
    Base64.encode64(
      {
        "expiration" => 1.hour.from_now.utc.xmlschema,
        "conditions" => [
          { "bucket" =>  GLOBAL[:aws_bucket] },
          [ "starts-with", "$key", "" ],
          { "acl" => "public-read" },
          [ "starts-with", "$Content-Type", "" ],
          [ "content-length-range", 0, 10 * 1024 * 1024 ]
        ]
      }.to_json).gsub(/\n/,'')
  end

  def s3_upload_signature
    Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest::Digest.new('sha1'), GLOBAL[:aws_secret], s3_upload_policy)).gsub("\n","")
  end
```

`s3_access_token` will be called when the client makes a GET request to the server, rendering a JSON response that contains the policy, signature and the public key. Notice that the Angular directive only specifies the bucket name, abstracting away the AWS keys, so you have to manually set the keys yourself on the server-side using environment variables.

If you’re using Rails, set the route as get `'/s3/upload_image' => 'users#s3_access_token', defaults: {format: 'json'}` in the `routes.rb` and you’re good to go. Notice that I’m using a `UsersController` here, you can substitute it with whatever controller you wish to use.

Just a caveat though, since the Angular directive uses Twitter Bootstrap 2 for the progress bar (yes it has a progress bar to indicate loading progress, how cool is that). The bar won’t be displayed when you are using Bootstrap 3. Hence you have to open up the source code of the directive and change `<div class="bar"` to `<div class="progress-bar"`.

Even though there exists cleaner solutions like paperclip gem for Ruby, I believe their solution is entirely server-side; their HTML form is served using ERB. If you’re looking for a solution that decouples the front-end and back-end, ng-s3upload is a good alternative.