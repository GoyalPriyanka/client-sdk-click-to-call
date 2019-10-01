# Client SDK - Call from a Webpage
Use case that demonstrates how to use a button on a webpage to dial a Nemxo number and have that routed to a dedicated support number.

## Purchase a Nexmo number

If you don't already have one, you can purchase a Nexmo number using either the [Dashboard](https://dashboard.nexmo.com) or the following [nexmo-cli](https://github.com/Nexmo/nexmo-cli) command:

```sh
nexmo number:buy -c GB --confirm
```

## Create an application

Run the following Nexmo CLI command in your application directory, replacing `abc123.ngrok.io` with your own server URL:

```sh
nexmo app:create ClickToCall https://abc123.ngrok.io/webhooks/answer https://abc123.ngrok.io/webhooks/event --keyfile=private.key --type=voice

Application created: APPLICATION_ID
No existing config found. Writing to new file.
Credentials written to /Users/mlewin2/Repos/nexmo/client-sdk-click-to-call/.nexmo-app
Private Key saved to: private.key
```

Make a note of the `APPLICATION_ID` returned by this command.

The `private.key` and `.nexmo-app` files are saved to your application directory.

## Link your Nexmo Number

Link your Nexmo number to your application:

```sh
nexmo link:app NEXMO_NUMBER APPLICATION_ID
```

## Create a User

Create a user that your site will use to place the call, with the following Nexmo CLI command:

```sh
nexmo user:create name="supportuser"
User created: USER_ID
```

Make a note of the `USER_ID` returned by this command.

## Generate a JWT

Generate a JWT that the `supportuser` will authenticate with, replacing `APPLICATION_ID` with the application ID you created earlier:

```sh
nexmo jwt:generate ./private.key sub=supportuser exp=$(($(date +%s)+86400)) acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{}}}' application_id=APPLICATION_ID 
```

Make a note of the JWT generated by this command. It expires after one day, after which you will need to regenerate it.

## Configure the application

Copy `example.env` to `.env` and then populate it as follows:

```
PORT=3000
JWT= /* The JWT */
NEXMO_NUMBER= /* The Nexmo Number associated with your application */
DESTINATION_PHONE_NUMBER= /* A target number to receive calls on */
```

Both phone numbers should include the country code, but omit any leading zeroes.

## Install dependencies

Run the following to install dependencies:

```sh
npm install
```

## Ensure that your webhooks are accessible

Your webhooks (in `server.js`) must be accessible over the public Internet for Nexmo's APIs to be able to notify you about inbound calls and associated events. Consider using a tool like [ngrok](https://ngrok.com) for this.

## Start your server

```sh
npm start
```

## Run the application

Launch your application from your browser by entering `http://localhost:3000` in the address bar (changing the port from `3000` to whatever you configured in `.env`).

Click the "Call Now!" button. The application should read a welcome message and then transfer the call to your target phone number (`DESTINATION_PHONE_NUMBER` in `.env`).


