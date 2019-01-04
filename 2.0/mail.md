# Mail

Adonis mail provider makes it super easy to send emails from your application. It makes use of node-mailer and has support for `SMTP`, `Amazon SES` and `Mandrill` out of the box.

Adonis mail also comes with a `Log` driver, which can be used to test your emails in the development environment.

- [Installing](#installing)
- [Drivers](#drivers)
- [Basic Usage](#basic-usage)
- [Sending emails](#sending-emails)
- [Building email message](#building-email-message)
- [Override config](#override-config)
- [Switching drivers](#switching-drivers)
- [Adding driver](#adding-driver)
- [Testing emails](#testing-emails)


## Installing

To setup mail provider, you are required to install and register it inside your `bootstrap/app.js` file.

```bash,line-numbers
npm i --save adonis-mail-provider
```

Now you need to register it inside the providers array and create an alias for same.

#### bootstrap/app.js
```javascript,line-numbers
const providers = [
 'adonis-mail-provider/providers/MailProvider'
]

const aliases = {
  Mail: 'Adonis/Addons/Mail'
}
```

## Drivers
Mail provider does not install direct dependencies required by mail drivers, and you must install them when you want to use a particular driver.

#### Mandrill
Mandrill requires [got](https://www.npmjs.com/package/got) to make request to Mandrill API in order to send emails. Make sure to install it when you are using the Mandrill driver.

```bash,line-numbers
npm i --save got
```

#### Amazon SES
Amazon SES driver is dependent upon [nodemailer ses transport](https://github.com/andris9/nodemailer-ses-transport), and you are required to install it before sending emails using SES driver.

```bash,line-numbers
npm i --save nodemailer-ses-transport
```

## Basic Usage

Once you have registered mail provider, you are ready to make use of it. Make sure to create the configuration file inside the config directory. The path of the file will be `config/mail.js`. You can also download the sample config file from [github](https://github.com/adonisjs/adonis-mail/blob/master/examples/mail.js).

```javascript,line-numbers
const Mail = use('Mail')

yield Mail.send('emails/welcome', {} , function (message) {
  message.to('someone@example.org')
  message.from('me@example.org')
  message.subject('Welcome to Adonis')
})
```

That's all you need to do to send emails.

## Sending Emails

Sending emails is a seamless process and remains same for all drivers. Wherever possible, drivers will create connections pool to re-use the same connection again.

#### send <span>(view, [data], callback, [configKey])</span>

```javascript,line-numbers
yield Mail.send('receipt', {name: 'doe', amount: 22}, function (message) {

})
```

1. **view (required)** - path to view to be used for email body.
2. **data (optional)** - data to pass to view
3. **callback (required)** - callback to build the message body
4. **configKey (optional)** - runtime configuration key to be used for sending an email.

#### setting up plain text body

`send` method accepts a string or an array of views to be used as email body. If you want to define different view for plain text, you can pass it as 2nd option inside an array

```javascript,line-numbers
yield Mail.send(['welcome', 'welcome.text', 'welcome.watch'])
```

1. **welcome** - html body for email.
2. **welcome.text** - plain text body for email.
3. **welcome.watch** - Apple watch html for email body (experimental).

#### raw <span>(body, callback, [configKey])</span>

```javascript,line-numbers
yield Mail.raw('Hello', function (message) {

}, configKey)
```

1. **body (required)** - body to be used as email body
2. **callback (required)** - callback to build the message body
3. **configKey (optional)** - runtime configuration key to be used for sending an email.

## Building email message

The email message is created using the argument passed to the callback, and following methods can be used.

#### from <span>(email, [name])</span>

```javascript,line-numbers
message.from('doe@example.com', 'John Doe')
```

#### sender <span>(email, [name])</span>
An e-mail address that will appear on the `Sender:` field.

```javascript,line-numbers
message.sender('doe@example.com', 'John Doe')
```

#### replyTo <span>(email, [name])</span>

```javascript,line-numbers
message.replyTo('doe@example.com', 'John Doe')
```

#### to <span>(email, [name])</span>

```javascript,line-numbers
message.to('jane@example.com', 'Jane Doe')
```

#### cc <span>(email, [name])</span>

```javascript,line-numbers
message.cc('jamie@example.com', 'Jamie Doe')
```

#### bcc <span>(email, [name])</span>

```javascript,line-numbers
message.bcc('me@example.com', 'Me')
```

#### subject <span>(message)</span>

```javascript,line-numbers
message.subject('Welcome to Adonis')
```

#### priority <span>(level)</span>
Email priority to be set for email. It can be one of the followings.

- high, low, normal

```javascript,line-numbers
message.priority('high')
```

#### header <span>(key, value)</span>
```javascript,line-numbers
message.header('x-id', 1)
```

#### headers <span>(headers=Array)</span>
```javascript,line-numbers
message.header([{key: 'x-id', value: 1}])
```

#### attach <span>(filePath, [options])</span>

```javascript,line-numbers
message.attach(__dirname, '/assets/logo.png')
// or
message.attach(__dirname, '/assets/logo.png', {filename: 'MyLogo.png'})
```

options to the `attach` method are optional and can be one of the following

| key | type | description
| ---- | ----- | ---------
| filename | String | name of the file. If not defined, will be created from the file path.
| contentType | String | attachment content type. If not defined, will be picked from the file extension.
| contentDisposition | String | content disposition, defaults to `attachment`.
| encoding | String | attachment encoding, can be `base64`, `hex`, `binary`.


#### attachData <span>(data, filename, [options])</span>

attach raw data as an attachment to the email.

```javascript,line-numbers
message.attachData('some raw content', 'raw.txt')
```
options are same as `attach` options.

#### embed <span>(filePath, cid, [options])</span>

attach a file to be used as inline attachments.

```javascript,line-numbers
messsage.embed(__dirname, '/assets/logo.png', 'unique-cid')
```

and later you can use the defined cid for attaching inline images.

```html,line-numbers
<img src="cid:unique-cid" />
```

#### html <span>(body)</span>
HTML for the email body is automatically defined by `send` method, if for any reasons you want to override it, make use of this method.

#### text <span>(body)</span>
text body can also be defined with the `send` method.

#### watchHtml <span>(body)</span>
watch html  can also be defined with the `send` method.

## Override config

Drivers configured with mail provider will use the default configuration defined in `config/mail.js` file. You can tell them to use different configuration by passing the configuration key to send method.

```javascript,line-numbers
yield Mail.send(view, data, callback, 'alternate.config')
```

<div class="__note"> Drivers will make the decision on whether or not to use the alternate configuration.</div>

## Switching drivers

Mail provider makes use of driver defined inside `config/mail.js` file. But also gives you an opportunity to use a different driver on runtime.

```javascript,line-numbers
const mandrill = Mail.driver('mandrill')
yield mandrill.send('..')
// or
yield Mail.driver('mandrill').send('....')
```

## Adding driver

You can extend mail provider by adding a new driver. As mail provider makes use of nodemailer, it will nice if you make use of node mailer transport and wrap it as a driver. Also, there are no hard rules on making use of node mailer.

#### Usage with node mailer transport

```javascript,line-numbers
class MailGun {
 
  constructor (Config) {
    this.config = Config
    this.transport = this._createTransport('mail.mailgun')
  }

  _createTranport (configKey) {
    const options = this.config.get(configKey)
    const nodemailer = require('nodemailer')
    const mailgunTransport = require('nodemailer-mailgun-transport')
    return nodemailer.createTransport(mailgunTransport(options))
  }

  send (message, configKey) {
    let transport = this.transport
    /**
     * here we make sure to entertain alternate configuration
     * sent with send method.
     */
    if (configKey) {
      transport = this._createTransport(configKey)
    }
    return transport.sendMail(message)
  }
}
```

Every driver should have a `send` method, and it should return a Promise, and that is the only requirement for a driver. 

Finally, we will add our driver to the Ioc container and specifying that we want to extend the mail provider. Same can be done inside `bootstrap/start.js`.

```javascript,line-numbers
App.on('start', function () {
 
  Ioc.extend('Adonis/Addons/Mail', 'mailgun', function (app) {
    const Config = app.use('Adonis/Src/Config')
    return new MailGun(Config)
  })

})
```

## Testing emails

It can become tedious to write tests for emails as they will be sent out to actual email addresses. To make it easier, mail providers comes with a `log` driver. It will write emails to a log file instead of sending them to the mail server.

By default emails will be written to `storage/logs/mail.eml`. You can also read emails from this file to make sure they are formatted correctly.

#### config/mail.js

```javascript,line-numbers
module.exports = {
    driver: 'log',
    log: {
        toPath: Helpers.storagePath('logs/mail.eml')
    }
}
```