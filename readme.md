
# How to create a website with a custome domain on AWS S3 from the commandline

This short tutorial assumes you have first [downloaded the AWS commandline (CLI) tools](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [set it up with your AWS credentials](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). Once complete you should see a folder called .aws in your home folder, with your credentials in it.

## Create a S3 bucket

To create a S3 bucket to store your site you must give it a unique name (if you don't it will return a warning to say that is already in use). I am going to use "seed-site"

    aws s3api create-bucket --bucket seed-site

It should return:

    { "Location": "/seed-site" }

If you got to your [AWS console](https://console.aws.amazon.com/s3) you should now see your new bucket.

![AWS-S3-screenshot](https://github.com/brianom/aws-create-s3-website-commandline/blob/master/images/AWS-S3-screenshot.png)

Create your website files and go to that folder. (you can use my index.html and other files available [here](https://github.com/brianom/aws-create-s3-website-commandline)). You can copy the files one at a time like this:

# Upload files
You can use cp a lot like the regular commandline copy.

    aws s3 cp index.html s3://seed-site/

returns

    upload: ./index.html to s3://seed-site/index.html

Or all the files in your folder and sub folders using a recursive copy. The command line parameters I've used tell it to just copy the jpeg and html files I need. I have also made all the files publically available using "--acl public-read"

    aws s3 cp . s3://seed-site/ --recursive --exclude "*" --include "*.jpg" --include "*.html" --acl public-read

You can verify they have been copied to the remote folder using a remote list.

    aws s3 ls s3://seed-site/

Returns:

    2017-06-07 06:38:29     329903 dead-seed.jpg
    2017-06-07 06:38:29       1608 error.html
    2017-06-07 06:38:29       1587 index.html
    2017-06-07 06:38:29     201254 seed.jpg

# Make it a website

Finally to configure this S3 bucket to serve the files as a website, run the following command

     aws s3 website s3://seed-site/ --index-document index.html --error-document error.html

Ta-Da: https://seed-site.s3.amazonaws.com/index.html
