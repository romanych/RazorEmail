##RazorMail

Razor Email is a email templating library that uses the RazorEngine for templating services.

You specify your email in an xml file, e.g. ForgotPassword.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<email>
	<subject>Reset Password Request @Model.Link</subject>
	<from display="jobping">noreply@jobping.com</from>
	<cc />
	<bcc />
</email>
```

The xml is used to specify the template for your email. Email bodies can be added inline (into the xml) or as external files.

By default the engine will look for an external razor templates..

 - A html view called: ForgotPassword.text_html.cshtml 
 - A text view called: ForgotPassword.text_plain.cshtml

Sending an email looks like this: 

```cs
RazorMailer.Build("ForgotPassword",  new { Link = "http://www.jobping.com" },"john.doe@test.com", "John Doe")
	.ToMailMessage()	
	.Send();
```

Or a more involved example: 

```cs
var mySampleModel = new
{
	Link = "http://www.jobping.com",
	RecentActivity = new List<Tuple<DateTime, string>>
	{
		{Tuple.Create(new DateTime(2009,7,4,16,49,23), "Signed up to Jobping")},
		{Tuple.Create(new DateTime(2010,1,13,16,49,23), "Created an api toke")},
		{Tuple.Create(new DateTime(2011,4,3,16,49,23), "Forgot your password & we sent a reset link to xyz@abc.com")},
		{Tuple.Create(new DateTime(2020,2,12,16,49,23), "Found a bug with the date")}
	}
};

RazorMailer .Build("ForgotPassword", mySampleModel,"john.doe@test.com", "John Doe")
	.WithHeader("X-RazorMail-Send-At", DateTime.Now.ToLongTimeString())
	.ToMailMessage()
	.SendAsync( (x, m) =>
	{
		Console.WriteLine(x);
		Console.WriteLine("Message Subject: {0}, Send around: {1}", m.Subject, m.Headers["X-RazorMail-Send-At"]);
	}, 
	"Sent John Doe his forgot password message");

```

By default, templates are picked up from the directory specific from the `razor.email.base.dir` AppSetting.

Since version 1.1.1.40, `IEmailResolver` interface was created to allow you to fully customize the loading of email templates (e.g. from a DB or otherwise). It is also possible to implement `ITemplateResolver` for more fine-grained control. These interfaces and the `RazorMailer.RazorMailer(ITemplateResolver, IEmailResolver)` ctor make dependency injection possible. 

Also as of 1.1.1.40, `SmtpMailSender` implements `IEmailSenderAsync` which uses the TPL and returns a `Task`.

There is a console app sample and a test project for further reference. 
Nuget: http://www.nuget.org/packages/RazorEmail/
