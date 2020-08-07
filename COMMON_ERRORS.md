# Common errors in Visual Studio/C#

## Make sure that the controller has a parameterless constructor
This error usually happens when you have injected a class into a constructor but the class/interface has not been registered in a container factory.

## Unauthorized (401) when trying to restore NuGets
This is usually caused by credential settings in the Credential Manager. Look in Windows Credentials and look for VSCredentials.