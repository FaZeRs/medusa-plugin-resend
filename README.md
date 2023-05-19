# medusa-plugin-resend

Resend email plugin to send transactional emails for Medusa using local Handlebars templates or local React templates

[Documentation](https://pevey.com/medusa-plugin-resend)

If you are not familiar with Medusa, you can learn more on [the project web site](https://www.medusajs.com/).

> Medusa is a set of commerce modules and tools that allow you to build rich, reliable, and performant commerce applications without reinventing core commerce logic. The modules can be customized and used to build advanced ecommerce stores, marketplaces, or any product that needs foundational commerce primitives. All modules are open-source and freely available on npm.

## Features

- Templates are stored locally.  
- Templates can either be handlebars templates or compiled React templates
- You can refer to the Medusa template reference to see all data fields that are available for each event: [Template Reference](https://docs.medusajs.com/plugins/notifications/sendgrid#template-reference)
- An API endpoint that is useful for testing and that can be used with other (non-Medusa) portions of your storefront application is included.  By default, the endpoint does nothing for security reasons.  See configuration options below to enable it.

## Node v20

- If you are starting to test out Node v20, be sure you give runtime permission for fs reads due to the new Node permissions API.  Otherwise, this plugin will not be able to read your email templates from the file system.

## Configuration

Enable in your medusa-config.js file similar to other plugins:

```bash
  {
    resolve: `medusa-plugin-resend`,
    options: {
      api_key: process.env.RESEND_API_ID,
      from: process.env.SES_FROM,
      enable_endpoint: process.env.SES_ENABLE_ENDPOINT,
      template_path: process.env.SES_TEMPLATE_PATH,
		subject_template_type: process.env.RESEND_SUBJECT_TEMPLATE_TYPE,
		body_template_type: process.env.RESEND_BODY_TEMPLATE_TYPE,
      order_placed_template: 'order_placed',
      order_shipped_template: 'order_shipped',
      customer_password_reset_template: 'customer_password_reset',
      gift_card_created_template: 'gift_card_created',
      //order_canceled_template: 'order_canceled',
      //order_refund_created_template: 'order_refund_created',
      //order_return_requested_template: 'order_return_requested',
      //order_items_returned_template: 'order_items_returned',
      //swap_created_template: 'swap_created',
      //swap_shipment_created_template: 'swap_shipment_created',
      //swap_received_template: 'swap_received',
      //claim_shipment_created_template: 'claim_shipment_created',
      //user_password_reset_template: 'user_password_reset',
      //medusa_restock_template: 'medusa_restock',
    }
  },
```

- Obtain an API key from you Resend dashboard

- The 'from' setting should be a valid email address on a domain you have properly configured with Resend

- The template path is the location of your email templates.  It can be a full (absolute) path or a path relative to the medusa root folder, e.g., 'medusa/data/templates'.

- The 'subject_template_type' can be 'handlebars' or 'text'.  The default is handlebars.  If you choose text, you cannot include dynamic values (like customer name or order number) in your subject lines.

- The body_template-type' can be 'handlebars' or 'react'.  The default is handlebars.  See below for info on using React templates.

## Why Resend?

There are 3 primary options for sending email with Medusa:

### Sendgrid

Uses Handlebars-style templates

Pros:
- Has a web-based GUI template editor
- Easy and quick to sign up and get started

Cons:
- Not the cheapest option
- Templates are not stored locally and not easy to add to version control
- Some people don't want a GUI email editor and find it a hindrance

### AWS Simple Email Service (SES)

Uses Handlebars-style templates (React and SvelteKit template options are coming soon)

Pros:
- The cheapest option.  Often free if you host your app on EC2.  Still usually the cheapest option even if you don't.
- Convenient if you are already using AWS services
- Medusa plugin uses local email templates that can be added to version control repos

Cons:
- No GUI email editor
- Some people find AWS difficult to navigate/set up
- New customers are placed in sandbox mode to start and have to submit a request to get real email-sending privileges

### Resend

Uses Handlebars OR React templates

Pros:
- Easy and quick to set up
- Medusa plugin uses local email templates that can be added to version control repos
- Can use React templates

Cons:
- No GUI email editor
- Not the cheapest option
- Currently in beta, so invite-only (or join the waitlist)

## Templates

The templates used are stored locally.  Create a 'data/templates' folder and include the entire path in the SES_TEMPLATE_PATH variable.

```bash
medusa-server  // root directory
|-data
	|-templates
		|-order_placed  // or whatever you name your templates and specify in the config file
			|-subject.hbs
			|-subject.text
			|-html.hbs
			|-html.jsx  // uncompiled React template
			|-html.js   // compiled React template
			|-text.hbs
		|- etc   
```

Not all of the above files are required.  You only need either subject.hbs or subject.text, depending on whether you have opted to use text or handlebars subject type.  If you have opted to use React body templates, you do not need html.hbs or text.hbs.  If you have opted to use Handlebars body templates you do not need the React templates.  You can use either html.hbs or text.hbs, or both.  If you include both, both templates will be compiled and sent, and it will be up to the user's email client which gets rendered.

## Transpiling React Templates

Inside your template folder (NOT your main medusa server folder), install babel:

```bash
npm i -D @babel/core @babel/cli @babel/preset-env @babel/preset-react
```

Example of compiling a template:

```bash
npx babel 'order_placed/html.jsx' --presets=@babel/preset-env,@babel/preset-react -o 'order_placed/html.js'
```

More options and convenient tooling for compiling Medusa email templates are in the works, including Tailwind CSS processing.

## Testing

This plugin adds an endpoint at http://[server]/resend/send

By default, the endpoint will refuse to send any emails.
This endpoint may be useful for testing purposes in a development environment or for use by related applications.

There is NO SECURITY on the endpoint by default. Most people will NOT need to enable this endpoint.
If you are certain that you want to enable it and that you know what you are doing,
set the environment variable RESEND_ENABLE_ENDPOINT to "42" (string).
The unsual setting is meant to prevent enabling by accident or without thought.
To use the endpoint, POST a json req.body with: template_id, from, to, and data to /ses/send.

## Acknowledgement

This plugin borrows extensively from medusa-plugin-sendgrid by Oliver Juhl.
