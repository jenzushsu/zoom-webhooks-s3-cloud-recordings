# Zoom Cloud Recordings S3 Auto-Upload with Zoom Webhooks

This Express/Node.js project demonstrates how to set up and listen to Zoom Webhooks that, when processed, will automatically stream a download and subsequent upload to AWS S3 for Cloud Recordings storage.

## Preparing Your AWS Account

This will outline the necessary steps to prepare your AWS account for use with this sample application.

### Setting up an S3 bucket for Zoom Cloud Recordings

In the AWS region of your choice, create a General purpose S3 bucket, naming it anything of your choosing. For this page, we'll use zoom-webhooks-cloud-recording-uploads as our S3 bucket name.

Once the bucket has been created, fetch the virtual-hosted-style path of the bucket. Using the bucket name above, zoom-webhooks-cloud-recording-uploads, our fully-qualified virtual host path would be:

```
https://zoom-webhooks-cloud-recording-uploads.s3.ap-southeast-1.amazonaws.com
```
Make note of the bucket URL, as it will be used in the future when configuring the sample application.

### Setting up an IAM policy that defines access to your Cloud Recording S3 bucket

Under IAM > Policies > Create policy in the AWS console, create a new policy that will deny access to everything except s3:PutObject into the S3 bucket that was created previously. Using the S3 bucket name defined above, zoom-webhooks-cloud-recording-uploads, the policy JSON will look like:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "UploadUserDenyEverything",
      "Effect": "Deny",
      "NotAction": "*",
      "Resource": "*"
    },
    {
      "Sid": "UploadUserAllowPutObject",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::zoom-webhooks-cloud-recording-uploads/*"
      ]
    }
  ]
}
```

### Setting up an IAM user with an Access Key ID and Secret Access Key, allowing programmatic access to AWS services

Under IAM > Users > Create user in the AWS console, create a new user that has the attached permission policy defined previously. When the user has been created, create an access key for the user, taking note of the Access Key ID and Secret Access Key, as those credentials will be used in this sample application to access AWS resources in a future section.

## Installation

In a terminal window (e.g., Git Bash for Windows or Terminal for Linux/Mac OS), clone this repository by executing the following command:

```bash
$ git clone https://github.com/zoom/zoom-webhooks-s3-cloud-recordings-uploader.git
```

## Setup & Configuration

1. In a terminal window, `cd` into the cloned repository:

    ```bash
    $ cd zoom-webhooks-s3-cloud-recordings-uploader
    ```

2. Install all necessary dependencies with `npm`, `yarn`, or `pnpm`:

    ```bash
    $ npm install
    ```

3. Rename `.env.local` to `.env`, replacing all environment variables for use with the AWS S3 and [Zoom's Webhooks](https://developers.zoom.us/docs/api/webhooks/).

4. Start the development server

    ```bash
    $ npm run dev
    ```

5. Once the server is up and running, Zoom requires all webhook endpoints are first validated before webhooks are sent. Refer to Zoom's [_Using Webhooks_](https://developers.zoom.us/docs/api/rest/webhook-reference/) guide for more information.

> [NOTE]
> The only required webhook event for this application is [**Meetings > Recording completed**](https://developers.zoom.us/docs/api/meetings/events/#tag/recording/POSTrecording.completed). All others events that are sent to this application will return `200 OK`, but will not be processed.

## Usage

This application exposes the `POST /` endpoint; however, this endpoint should not be called manually. Instead, this endpoint is called by Zoom once a recording has finished successfully.

## Deployment

As this application is written in TypeScript, it will need to be deployed with the ability to run via `ts-node`, `tsx`, `swc-node`, or similar; `node` will be unable to run this application out of the box.

If you want to be able to run the application via the `node` command, it must first be built by executing `npm run build`, which will transpile all TypeScript files to JavaScript, outputting them to the `dist` directory. Once transpiled you can run the application by executing `node server.js` within the `dist` directory.