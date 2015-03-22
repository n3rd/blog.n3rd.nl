---
layout: post
title: Automatic sprite images for MonoRail
tags: [ASP.NET, MonoRail]
description:
---

![sprite](https://cloud.githubusercontent.com/assets/3578344/6769845/fa05e77a-d0aa-11e4-937e-934303e8220e.png){: .callout}

Almost a year ago [Scott Hanselman](http://www.hanselman.com/blog/NuGetPackageOfTheWeek1ASPNETSpriteAndImageOptimization.aspx) blogged about a NuGet package called AspNetSprites that automatically generates sprite images for you. When using a lot of small icon images, sprites can speedup your website quite a lot. This is due to minimizing the number of HTTP requests.

For AspNetSprites there are helper packages for both WebForms and ASP.NET MVC, but wouldn't it be nice to use this with MonoRail too? This is not to difficult to accomplish because there is a AspNetSprites-Core package containing all the logic so all we need to do is write a MonoRail View Helper class.

I downloaded the source from the ASP.NET [codeplex](http://aspnet.codeplex.com/) page and modified the Razor view helper so it can be used with MonoRail. Here the result:

{% highlight c# %}
using System;  
using System.Linq;  
using System.Text;  
using Microsoft.Web.Samples;  
using System.IO;  
using System.Web;  
using System.Web.UI;  
using System.Collections;  

namespace MonoRail.Samples.SpriteHelper  
{  
    public class Sprite  
    {  
        private static Control helperControl = CreateHelperControl();  

        public static string ImportStylesheet(string virtualPath)  
        {  
            ImageOptimizations.EnsureInitialized();  

            if (Path.HasExtension(virtualPath))  
            {  
                virtualPath = Path.GetDirectoryName(virtualPath);  
            }  

            HttpContextBase httpContext = new HttpContextWrapper(HttpContext.Current);  

            string cssFileName = ImageOptimizations.LinkCompatibleCssFile(httpContext.Request.Browser) ?? ImageOptimizations.LowCompatibilityCssFileName;  

            virtualPath = Path.Combine(virtualPath, cssFileName);  
            string physicalPath = HttpContext.Current.Server.MapPath(virtualPath);  

            if (File.Exists(physicalPath))  
            {  
                StringWriter sw = new StringWriter();  

                using (HtmlTextWriter html = new HtmlTextWriter(sw))  
                {  
                    html.AddAttribute(HtmlTextWriterAttribute.Href, ResolveUrl(virtualPath));  
                    html.AddAttribute(HtmlTextWriterAttribute.Rel, "stylesheet");  
                    html.AddAttribute(HtmlTextWriterAttribute.Type, "text/css");  
                    html.AddAttribute("media", "all");  
                    html.RenderBeginTag(HtmlTextWriterTag.Link);  
                }  

                return sw.ToString();  
            }  

            return String.Empty;  
        }  


        public static string MakeCssClassName(string pathToImage)  
        {  
            return ImageOptimizations.MakeCssClassName(pathToImage);  
        }  

        public static string Image(string virtualPath)  
        {  
            return Image(virtualPath, null);  
        }  

        public static string Image(string virtualPath, IDictionary htmlAttributes)  
        {  
            ImageOptimizations.EnsureInitialized();  

            HttpContextBase httpContext = new HttpContextWrapper(HttpContext.Current);  

            StringWriter sw = new StringWriter();  

            using (HtmlTextWriter html = new HtmlTextWriter(sw))  
            {  
                if (htmlAttributes != null)  
                {  
                    foreach (DictionaryEntry entry in htmlAttributes)  
                    {  
                        html.AddAttribute((entry.Key ?? String.Empty).ToString(), (entry.Value ?? String.Empty).ToString());  
                    }  
                }  

                if (ImageOptimizations.LinkCompatibleCssFile(httpContext.Request.Browser) == null)  
                {  
                    html.AddAttribute(HtmlTextWriterAttribute.Src, ResolveUrl(virtualPath));  
                }  
                else  
                {  
                    html.AddAttribute(HtmlTextWriterAttribute.Class, ImageOptimizations.MakeCssClassName(virtualPath));  
                    html.AddAttribute(HtmlTextWriterAttribute.Src, ResolveUrl(ImageOptimizations.GetBlankImageSource(httpContext.Request.Browser)));  
                }  

                html.RenderBeginTag(HtmlTextWriterTag.Img);  
            }  

            return sw.ToString();  
        }  

        private static Control CreateHelperControl()  
        {  
            var control = new Control();  
            control.AppRelativeTemplateSourceDirectory = "~/";  
            return control;  
        }  

        private static string ResolveUrl(string path)  
        {  
            return helperControl.ResolveClientUrl(path);  
        }  

    }  
}  
{% endhighlight %}

This helper exposes three methods (and one overload) here are some usage examples:

`$Sprite.ImportStylesheet("~/App_Sprites/categories/")`

`<li class="$Sprite.MakeCssClassName("~/App_Sprites/categories/dotNet.png")"><a href="#" class="categories">Programming</a></li>`

`<a href="#">$Sprite.Image("~/App_Sprites/popular/visualStudio.png", "%{alt='visualStudio'}")
</a>`

Besides this code file a small change to the web.config file is required and you should off course add the [Helper(typeof(Sprite))] attribute to your controller.

To get you started here is the NuGet packages I created for personal usage:
[AspNetSprites-MonoRailHelper.0.1.nupkg](http://n3rd.nl/shit/AspNetSprites-MonoRailHelper.0.1.nupkg)
