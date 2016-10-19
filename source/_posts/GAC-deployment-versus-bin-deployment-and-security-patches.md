---
title: GAC Deployment versus BIN Deployment and security patches
date: 2012-10-12 21:20:25
tags: [.NET, COM, C#]
---

In one of my assignment I had to investigate different ways to publish utility libraries to different projects and development team. The first idea that came to my mind was to build a Nuget package and to configure an internal Nuget Feed where I could publish my package. This sound like a good idea and I was going to close the analysis phase and settle down for the implementation when someone came to me and asked a question about how I was going to manage the Security patch deployment. Let me clarify what is a security patch.

A Security patch is a patch that needs to be deploy to production no matter of the risk for the production application to gets into trouble. It is a patch that does not contain any API or interface change but contains only internal corrections. Those patch are not deployed in the scope of a particular application but are deployed on any machine a specific component is used. In my situation, as I’m only delivering Libraries, I need to be able to tell to the ops team “Please deploy this on all machine the library is in use”. And this is my major problem. I don’t know. My library get used through Nuget and only the client applications know the package they are using. I also cannot guarantee that a fix on one package with the publication of a new Nuget package version will be picked up right away by the client application development team and included in their next deployment.

What option do I have here? Nuget package do not cover this scenario by design. My first reaction was to challenge the requirement? What kind of library might require a security patch? Not so many. You know what they say: “Show me a dragon and then I will show you excalibur”. This did not convince. I had to find a specific way to deploy those security sensible libraries.

This when I started investigating GAC deployment. How do I achieve GAC deployment? Well I build my library and I make it available through an MSI. This MSI will register that library in the GAC. The MSI being deploy on machine as a unit of deployment, it can be tracked and inventoried by the OPS team. I can find the list of machine where I will have to deploy my security patch.

The GAC deployment provides me with the possibility to deploy a new version version of a library on a machine and make sure any client application using the old version of the component will pick up the new version right away. I tested this and this is how I achieved this:

I wrote two libraries with the following code:

## Version 1.0.0.0

```csharp

using System.IO;
 
namespace CommonLibrary
{
    public class Library
    {
        public void Function()
        {
            var file = new StreamWriter("D:\\VersionLog.txt",true);
            file.WriteLine("This is version 1.0.0.0 of the library's function");
            file.Close();
        }
    }
}

```

## Version 2.0.0.0

```csharp

using System.IO;
 
namespace CommonLibrary
{
    public class Library
    {
        public void Function()
        {
            var file = new StreamWriter("D:\\VersionLog.txt",true);
            file.WriteLine("This is version 2.0.0.0 of the library's function");
            file.Close();
        }
    }
}

```

Both of them where compiled and strongly signed with the same key.

I wrote a small client app that was using the library It looked like that:

```csharp

namespace ClientApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            var lib = new Library();
            lib.Function();
        }
    }
}

```

I built those components on .NET framework 4.5 meaning I’m using the GAC of the .NET framework 4.0.

I deploy the version 1.0.0.0 to the GAC using the following .NET framework 4.0 gacutil command from a visual studio 2012 command dos prompt:

```shell-session

gacutil /i CommonLibrary.dll

```

I could then use the following command to check the proper installation of my component in the GAC

```shell-session

gacutil /l CommonLibrary

```

And the result was:

```shell-session

Microsoft (R) .NET Global Assembly Cache Utility.  Version 4.0.30319.17929
Copyright (c) Microsoft Corporation.  All rights reserved.
 
The Global Assembly Cache contains the following assemblies:
  CommonLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=b523b0464e4856a6, processorArchitecture=MSIL
 
Number of items = 1

```
Then I ran the client application application exe file and I could check in my output file the following line

```shell-session

This is version 1.0.0.0 of the library's function

```

Then I deployed the version 2.0.0.0 of the library using the same method as previously mentioned.
I then run the the following command to check the content of my GAC

```shell-session

D:\dev\GacVersionning\libraries\V2>gacutil /l CommonLibrary
Microsoft (R) .NET Global Assembly Cache Utility.  Version 4.0.30319.17929
Copyright (c) Microsoft Corporation.  All rights reserved.
 
The Global Assembly Cache contains the following assemblies:
  CommonLibrary, Version=1.0.0.0, Culture=neutral, PublicKeyToken=b523b0464e4856a6, processorArchitecture=MSIL
  CommonLibrary, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b523b0464e4856a6, processorArchitecture=MSIL
 
Number of items = 2

```
This clearly shown the multiple version of my deployed library.
After running the client application I could see my client application was still using the version 1.0.0.0.
In order for my client application to use the version 2.0.0.0 of the library I have to deploy a policy file.
A policy file is a config file (XML) that gets compiled into a dll in order to be deployed to the gac.
This will tell the GAC to redirect all calls for a given version to a another version.

This is the content of my policy config file that I named RedirectPolicyFile.config.

```xml

<configuration>
    <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
            <dependentAssembly>
                <assemblyIdentity name="CommonLibrary" publicKeyToken="b523b0464e4856a6" culture="neutral" />
                    <bindingRedirect oldVersion="1.0.0.0" newVersion="2.0.0.0"/>
            </dependentAssembly>
        </assemblyBinding>
    </runtime>
</configuration>

```

I compiled it using the following command

```shell-session

al /link:RedirectPolicyFile.config /out:policy.1.0.CommonLibrary.dll /keyf:StrongName.snk

```

Then i registered the policy “policy.1.0.CommonLibrary.dll” to the gac using the same command as usual

```shell-session

gacutil /i policy.1.0.CommonLibrary.dll

```

We can then run the client application and check the output file. It should contain the following line:

```shell-session

This is version 2.0.0.0 of the library's function

```

You have been security patched.


