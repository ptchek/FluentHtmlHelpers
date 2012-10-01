#WRITING CUSTOM HTML HELPERS FOR ASP.NET MVC 
*Ed Charbeneau*

##TRANSITIONING FROM WEB FORMS

As a web forms developer, I found the transition to MVC to be a bit of a shock at first. Without fully understanding the nature of MVC, I found the lack of a Toolbox filled with server controls to be confusing. However, once it became clear that the goal of MVC was to expose HTML markup and give developers full control over what is rendered to the browser, I quickly embraced the idea.

In MVC development, HTML helpers replace the server control, but the similarities aren’t exactly parallel. Whereas web forms and server controls were intended to bring the workflow of desktop forms to the web, MVC's HTML helpers simply provide a shortcut to writing out raw HTML elements that are frequently used.

The HTML helper, in most cases, is a method that returns a string. When we call a helper in a View using the razor syntax **@Html**, we are accessing the Html property of the View, which is an instance of the HtmlHelper class.

Writing extensions for the **HtmlHelper** class will allow us to create our own custom helpers to encapsulate complex HTML markup. Custom helpers also promote the use of reusable code and are unit testable. Custom helpers can be configured by passing values to the constructor, via fluent configuration, strongly typing, or a combination of both, and ultimately return a string. 

##HOW TO BEGIN

The first step in writing an HTML helper is finding code within our project that you intend on reusing. For the extent of this article I will be using an **Alert** message as an example. The alert is a UI element that displays a message which has a default, success, warning, or information style. The alert element’s markup is simple in construction but gives us an adequate sample for the scope of this article.

![This is how the alert element looks in the browser.](./images/Alert-box.jpg)

With our code targeted, we’ll examine the alert markup and see how we can break it down and construct reusable pieces of content. The alert is made up of a div container, a message, a close button and can have multiple styles applied to it. If we examine the markup, we can see that there are parts of the element that are static, and parts that can be broken down into options that can be set as parameters. 

![Alert element markup details](./images/markup-details.jpg)

![Alert element broken down in to static markup and options](./images/parameter-breakdown.jpg)

*In addition to the HTML, the alert will require CSS and JavaScript in order to function. The CSS and JavaScript are beyond the scope of this article but are included in the final example at the end of this article.*
 
##WRITING A SPECIFICATION

Personally, once I have my HTML defined, I prefer to start writing a specification before I do anything else. Writing a specification isn’t a necessary step for creating a custom HTML helper, but it gives me a guide to how the HTML helper will function and what the desired syntax will be. A spec will also promote the use of semantic code, which improves discoverability for others that may be using the helper.

Using a text file, I spec out how the helper will function and, since I will be unit testing the code, I’ll save the spec to my test project. There are several parameters that will be required to configure the Alert element: the text to be displayed, the style of the alert, and a close button that can be toggled. I’ll also be giving the Alert a default configuration that will only require that the text parameter be set. In addition to the elements parameters, we’ll allow HTML attributes to be passed to the helper as well (which I'll return to once we have our basic implementation up and running). Each configuration of the **Alert** helper is written out in the spec so it can be followed during implementation.

AlertHelperSpec.html

    //Given the Alert HTML Helper method
    
    //Then Render the HTML
    
    //Default Alert
    @Html.Alert(text:"message") [.HideCloseButton()]
    
    <div class="alert-box">
    	Message
    	<a href="" class="close">×</a>
    </div>
    
    //Success Alert
    
    @Html.Alert(text:"message", style:AlertStyle.Success [,hideCloseButton:false ,htmlAttributes:object])
    
    <div class="alert-box success">
    	Message
    	<a href="" class="close">×</a>
    </div>
    
    //Warning Alert
    
    @Html.Alert(text:"message", style:AlertStyle.Warning [,hideCloseButton:false ,htmlAttributes:object])
    
    <div class="alert-box warning">
    	Message
    	<a href="" class="close">×</a>
    </div>
    
    //Info Alert
    
    @Html.Alert(text:"message", style:AlertStyle.Info [,hideCloseButton:false ,htmlAttributes:object])
    
    <div class="alert-box info">
    	Message
    	<a href="" class="close">×</a>
    </div>

##UNIT TESTING

ASP.NET MVC is highly regarded for its ability to be unit tested, and custom HTML helpers can be thoroughly tested too. With the right setup, unit testing your custom helper isn’t difficult: 

First, we need to create an instance of the **HtmlHelper** class so our extension method can be tested. Next, the custom method is called, and finally we can check the results against our expectations.

So, before we can write our test, we will need to create an instance of the **HtmlHelper** class. However, the **HtmlHelper** class has no default constructor, so a little work must be done up front to get an instance. To create an instance of **HtmlHelper**, we must specify a context and view data container - for the scope of this article, fakes will do just fine. Since each test will require an instance of **HtmlHelper**, I’ve created an **HtmlHelperFactory** class to create the instances.

###HtmlHelperFactory

Now that the **HtmlHelperFactory** is available, getting an instance of HtmlHelper is as simple as calling **HtmlHelperFactory.Create()**.

Using the first spec, I’ll create a unit test for the default alert. In this test, the **Alert** method will be called, and should return the HTML markup we defined, the message specified, with no additional style, and a visible close button. 

First, we arrange our expected output and create an instance of HtmlHelper:

    //Spec
    //Should render an default alert box
    //@Html.Alert(text:"message")
    //arrange
    string htmlAlert = @"<div class=""alert-box"">message<a class=""close"" href="""">×</a></div>";
    var html = HtmlHelperFactory.Create();

At this point, the method has not been defined yet, so we’ll use what has been defined in the spec to guide us:

    //act
    var result = html.Alert("message").ToHtmlString();

Finally, we check our results with the expected output using **Assert.AreEqual**:

    //assert
    Assert.AreEqual(htmlAlert, result, ignoreCase: true);

So now the first unit test is written, but before it can be put to use the **Alert** method must be created. This test-first approach ensures that the custom Helper we write gives us the result we defined as HTML in our spec and, as each spec is fulfilled, this process will be repeated until all the specs are completed and satisfied.


##BASIC IMPLEMENTATION

Before creating our implementation, there are a few things we should know about MVC and the **HtmlHelper** class. The **HtmlHelper** class provides methods that help you create HTML controls programmatically; all **HtmlHelper** methods generate HTML and return the result as a string.

We’ll begin by creating a new class and implementing the **IHtmlString** interface - this provides the **ToHtmlString** method, which is used by MVC to render the control to the View. Next we override the **ToString** method of our class; the **ToString** and **ToHtmlString** methods will return the same result, which is a common practice for **HtmlHelpers**.

    public class AlertBox : IHtmlString
    {
        private readonly string text;

        public AlertBox(string text)
        {
            this.html = html;
            this.text = text;
        }

        //Render HTML
        public override string ToString()
        {
            return "";
        }

        //Return ToString
        public string ToHtmlString()
        {
            return ToString();
        }
    }

Now that we have an **HtmlHelper** class, we need to be able to call it from MVC. We’ll do this by writing an extension method that returns our custom **HtmlHelper** class:

    /// <summary>
    /// Generates an Alert message
    /// </summary>
    public static class AlertHtmlHelper
    {
        public static AlertBox Alert(this HtmlHelper html, string text)
        {
            return new AlertBox(text);
        }
    }

At this point a complete scaffold of our code is complete, and our unit test should execute but fail to pass.

To finish our basic implementation and pass the unit test, we’ll need to set up our parameters and render the HTML. MVC provides the **TagBuilder** class for building HTML, which we’ll use to build our render method.

        private string RenderAlert()
        {

            var wrapper = new TagBuilder("div");
            wrapper.AddCssClass("alert-box");

            var closeButton = new TagBuilder("a");
            closeButton.AddCssClass("close");
            closeButton.Attributes.Add("href", "");
            closeButton.InnerHtml = "×";

            wrapper.InnerHtml = text;
            wrapper.InnerHtml += closeButton.ToString();

            return wrapper.ToString();
        }

        //Render HTML
        public override string ToString()
        {
           return RenderAlert();
        }

The HTML helper should now pass the unit test.

With the basic implementation complete, we can easily expand on the HTML helper by adding additional options. Following our spec, we’ll add the option to change the style of the alert. Once again, we start with a unit test and then modify our code to complete the test:

    [TestMethod]
    public void ShouldCreateSuccessAlert()
    {
	    //Spec 
	    //Should render a Success alert box
	    //@Html.Alert(text:"message", style:AlertStyle.Success)
	    //arrange
	    string htmlAlert = @"<div class=""alert-box success"">message<a class=""close"" href="""">×</a></div>";
	    var html = HtmlHelperFactory.Create();
	
	    //act
	    var result = html.Alert("message", AlertStyle.Success).ToHtmlString();
	
	    //assert
	    Assert.AreEqual(htmlAlert, result, ignoreCase: true);
    }



    public class AlertBox : IHtmlString
    {

    private readonly string text;

    private readonly AlertStyle alertStyle;
 
    private readonly bool hideCloseButton;

    public AlertBox(HtmlHelper html, string text, AlertStyle style, bool hideCloseButton)
         {
            this.html = html;
            this.text = text;
            this.alertStyle = style;
            this.hideCloseButton = hideCloseButton;
         }
 
		private string RenderAlert()
        {
            if (alertStyle != AlertStyle.Default)
                wrapper.AddCssClass(alertStyle.ToString().ToLower());
            wrapper.AddCssClass("alert-box");
 
            //build html
            wrapper.InnerHtml = text;
            
            //Add close button
            if (!hideCloseButton)
                wrapper.InnerHtml += RenderCloseButton();
            
            return wrapper.ToString();
        }

        private static TagBuilder RenderCloseButton()
        {
             //<a href="" class="close">x</a>
             var closeButton = new TagBuilder("a");
             closeButton.AddCssClass("close");
             closeButton.Attributes.Add("href", "");
             closeButton.InnerHtml = "×";
             return closeButton;
        }
    }

Finally, we’ll make our helper more flexible by giving the end user the ability to define additional HTML attributes. The **TagBuilder**’s **MergeAttributes** method adds a specified attribute to the tag being rendered. In addition to the **MergeAttributes** method, the **HtmlHelper** **AnonymousObjectToHtmlAttributes** is used to allow an anonymous object to be used to define additional parameters:

    public class AlertBox : IHtmlString
    {...
    	private object htmlAttributes;
	
	    public AlertBox(string text, AlertStyle style, bool hideCloseButton = false, object htmlAttributes = null)
	
	    private string RenderAlert()
	    { ...
	    	wrapper.MergeAttributes(htmlAttributes != null ? HtmlHelper.AnonymousObjectToHtmlAttributes(htmlAttributes) : null);
	    ...}
    }

##FLUENT CONFIGURATION

Appending the spec

Method chaining with interfaces

More testing

##STRONGLY TYPED HELPERS

Appending the spec again

HtmlHelper extension methods again

The ElementFor convention

`<TModel>`

`Expression<Func<T,T>>`

Model Metadata

##FINAL RESULTS
Basic

Fluent

Strongly typed

##CONSIDERATIONS

When to create a helper

When to use basic, fluent, strongly typed or all.

What’s next, templates, and complex controls

Resources