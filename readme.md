
# Host a website with a custom domain on S3 from the commandline

This short guide assumes you have first [downloaded the AWS commandline (CLI) tools](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [set it up with your AWS credentials](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). 

It is also going to assume you register your domain with [Route53](https://console.aws.amazon.com/route53/) (the AWS DNS management service). This will save a bunch of hassle and only costs a little more.

For reference, you could just use the AWS console and follow [this guide](http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html), but cutting and pasting the following commands is way easier than navigating the AWS console.

I'm going to focus on just the custom domain part here, there is more detail post on creating the site from the commandline [here](https://github.com/brianom/aws-create-s3-website-commandline)

## Create a S3 bucket for the custom domain

For your new domain "example.com", you must create a bucket with the name the same as the domain name:

    aws s3 mb s3://example.com

To upload a local index.html file and give everyone in the world permission to read it (you can borrow mine [here](https://github.com/brianom/aws-s3-website-custom-domain)):

    aws s3 cp index.html s3://example.com --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers

You can visit your site here, http://example.com.s3.amazonaws.com/index.html

# Configure s3 bucket for hosting

First it's a good idea to create and upload an error page:

    aws s3 cp error.html s3://example.com --grants read=uri=http://acs.amazonaws.com/groups/global/AllUsers

Then set s3 to route traffic to them both:

     aws s3 website s3://example.com --index-document index.html --error-document error.html

# Configure Route53 to direct your domain to s3

If you have registered your domain with AWS it will automatically set up a "hosted zone" with the AWS name servers set up. So all we need to do is set up the Alias record which directs traffic to the domain to the s3 bucket.

We are going to use the aws route53 command change-resource-record-sets, to do this we need the hosted-zone-id of the domain and the region. The regions cannot be retrieved via the API, but are listed [here](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region), eu-west-ireland is Z1BKCTXD74EZPE.

To get the hosted-zone-id of the domain use:

    aws route53 list-hosted-zones

you are looking for a code that looks something like: Z2C72EK3L6ZJVX. you can see it here:

    "Id": "/hostedzone/Z2C72EK3L6ZJVX"

To set up the alias record, you can now plug these, your region and your domain in to the following command:

    aws route53 change-resource-record-sets --hosted-zone-id Z2LHOQSGWMPWH7 --change-batch '
    {
      "Comment": "Create an alias record to map my new domain to an s3 bucket",
      "Changes": [
        {
          "Action": "CREATE",
          "ResourceRecordSet": {
            "Name": "example.com.",
            "Type": "A",
            "AliasTarget": {
              "HostedZoneId": "Z1BKCTXD74EZPE",
              "DNSName": "s3-website-eu-west-1.amazonaws.com.",
              "EvaluateTargetHealth": false
            }
          }
        }
      ]
    }'

The response should look like:

    {
        "ChangeInfo": {
            "Status": "PENDING", 
            "Comment": "Create an alias record to map my new domain to an s3 bucket", 
            "SubmittedAt": "2017-06-14T21:29:56.153Z", 
            "Id": "/change/C1NNTHU82J8NBS"
        }
    }

It is generally a good idea to repeat these steps for the www.example.com sub domain.
