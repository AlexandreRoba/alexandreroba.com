---
title: Isolated calls of dynamically loaded assemblies
date: 2012-08-27 21:20:25
tags: [C#, .NET]
---

## Definition of the issue:

In one of the project I’m currently working on, I need to be able to call a function from an assembly that will be provided at run time. 
One of the major requirement I have is to have a clear isolation of the call with a minimum of configuration. 
The second requirement is  to be able to provide a regular configuration file on my callee assembly in order for a lambda developer to implement a WCF call in that assembly using regular config files. 
Meaning they should be able to write a simple .NET Assembly referencing other assembly and making use of a config file and all that should work.

There is no particular performance requirement. It is left to the developper of the callee assembly to manage this issue. It will be up to him to dispatch and manage threads if needed.

## My first solution:

The easiest solution I found was to create a new app domain. To load the callee assembly in that new domain and to execute the call there. This gave me the isolation level I was needing.

My calling class looks like this

```csharp
//Setting the new app domaine configuration
AppDomainSetup appDomainSetup = new AppDomainSetup();
appDomainSetup.ConfigurationFile = "Custom.Config"; //The config file of my calle assembly
appDomainSetup.ApplicationName = "ProxyName"; //Just to be cleaner
appDomainSetup.ApplicationBase = @"D:\dev\Dummy\ConsoleApplication2\ProxyComponent\bin\Debug\"; //Where is my calee assembly
//Creating a new app domain
AppDomain domain = AppDomain.CreateDomain("IsolatedDomain", null, appDomainSetup);
//My parameters
string dllFilePath = @"D:\dev\Dummy\ConsoleApplication2\ProxyComponent\bin\Debug\ProxyComponent.dll";
string proxyFullName = "ProxyComponent.Proxy";
//Loading my assembly in my new app domaine
IScheduler myProxy = (IScheduler)domain.CreateInstanceFromAndUnwrap(dllFilePath, proxyFullName);
//Executing my call on the new app domain
Console.WriteLine(myProxy.GetSettingValue("Key01"));

```
My ProxyComponent.Proxy class looks like this

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appsettings>
    <add key="Key01" value="Value01"/>
  </appsettings>
</configuration>

```
The only shared assembly bet ween the callee and the caller  is the assembly that contains the **IScheduler interface**.

```csharp
public interface IScheduler
{
	string GetSettingValue(string key);
}

```
This could have been avoided using DLR and Dynamic. I’ll try to work on this in a near future.


