# CommunityServer-CustomModules

## How it works

**1.** Get CommunityServer from https://github.com/ONLYOFFICE/CommunityServer 

**2.** Copy EmptySample and Sample to Products folder https://github.com/ONLYOFFICE/CommunityServer/tree/master/web/studio/ASC.Web.Studio/Products

**3.** Copy ASC.Api.Sample to ASC.Api folder https://github.com/ONLYOFFICE/CommunityServer/tree/master/module/ASC.Api

**4.** Add to build.proj https://github.com/ONLYOFFICE/CommunityServer/blob/master/build/msbuild/build.proj same code
```
<!-- Samples -->
<ProjectToBuild Include="$(ASCDir)web\studio\ASC.Web.Studio\Products\EmptySample\ASC.Web.EmptySample.csproj"/>
<ProjectToBuild Include="$(ASCDir)web\studio\ASC.Web.Studio\Products\Sample\ASC.Web.Sample.csproj"/>
```
and
```
<ProjectToBuild Include="$(ASCDir)module\ASC.Api\ASC.Api.Sample\ASC.Api.Sample.csproj"/>
```
by analogy with already existing projects
NOTE!!! the order is important

**5.** Add ASC.Api.Sample.SampleApi to web\studio\ASC.Web.Studio\web.autofac.config
```
<component
          type="ASC.Api.Sample.SampleApi, ASC.Api.Sample"
          service="ASC.Api.Interfaces.IApiEntryPoint, ASC.Api"
          name="sample"/>
```

**6.** run the Build.bat file https://github.com/ONLYOFFICE/CommunityServer/blob/master/build/Build.bat

## How to create your own module for ONLYOFFICE

**1.** Create an ASP.NET Web Application (ASC.Web.Sample) project
and put it to the ...web\studio\ASC.Web.Studio\Products\Sample folder
IMPORTANT!!! The output dll file name must be "ASC.Web.*.dll";

**2.** Connect the required references from ...\web\studio\ASC.Web.Studio\bin\
```
ASC.Common.dll
ASC.Core.Common.dll
ASC.Data.Storage.dll
ASC.Web.Core.dll
ASC.Web.Studio.dll
```

**3.** Implement the IProduct interface in the ProductEntryPoint.cs file
IMPORTANT!!! The ProductID must be unique Guid (in VS2012 is generated as TOOLS->GUID->New GUID)

**4.** NOTE!!! Add the following in the AssemblyInfo.cs file:
```
[assembly: Product(typeof(ASC.Web.Sample.Configuration.ProductEntryPoint))]
```

**5.** Inherit the Master from web\studio\ASC.Web.Studio\Masters\BaseTemplate.master

**6.** Set the output path in the project properties as
```
<OutputPath>..\..\bin\</OutputPath>
```
so that the builds were created at the web\studio\ASC.Web.Studio\bin folder

**7.** The project can be built manually or using the builder.
For the latter add the following lines to the build\msbuild\build.proj file:
```
<ProjectToBuild Include="$(ASCDir)web\studio\ASC.Web.Studio\Products\Sample\ASC.Web.Sample.csproj"/>
```
and run the build\Build.bat file

**8.** After the build run the website at the localhost:port address,
go to the "Modules & Tools" Settings page (http://localhost:port/management.aspx?type=2)
and enable the new Sample module.
It will be available in the portal header drop-down menu afterwards
or using the direct link: http://localhost:port/products/sample/default.aspx

## How to create API for your own module

**1.** Create an Class Library (ASC.Api.Sample) project
and put it to the ...module\ASC.Api\ASC.Api.Sample folder
IMPORTANT!!! The output dll file name must be "ASC.Api.*.dll";

**2.** Connect the required references from ...\web\studio\ASC.Web.Studio\bin\
```
ASC.Api.dll
ASC.Web.Sample.dll
```

**3.** Create SampleApi class and implement the IApiEntryPoint interface.
```
public class SampleApi : IApiEntryPoint
{
    public string Name
    {
        get { return "sample"; }
    }
}
```

**4.** Create public methods with specific attributes.
```
[Attributes.Create("create", false)]
public SampleClass Create(string value)
{
    return SampleDao.Create(value);
}
```
The attribute specifies the type of method, the path by which this method will be called, the authorization and verification of the tariff plan in it.
Possible options are shown below:
```
CreateAttribute(string path, bool requiresAuthorization = true, bool checkPayment = true) //corresponds to the "POST" request
UpdateAttribute(string path, bool requiresAuthorization = true, bool checkPayment = true) //corresponds to the "PUT" request
DeleteAttribute(string path, bool requiresAuthorization = true, bool checkPayment = true) //corresponds to the "DELETE" request
ReadAttribute(string path, bool requiresAuthorization = true, bool checkPayment = true) //corresponds to the "GET" request
```
the parameters requiresAuthorization, checkPayment are optional and have a value of true by default

**5.** Set the output path in the project properties as
```
<OutputPath>..\..\..\web\studio\ASC.Web.Studio\bin\</OutputPath>
<DocumentationFile>..\..\..\web\studio\ASC.Web.Studio\bin\ASC.Api.Sample.XML</DocumentationFile>
```
so that the builds were created at the web\studio\ASC.Web.Studio\bin folder

**6.** The project can be built manually or using the builder.
For the latter add the following lines to the build\msbuild\build.proj file:
```
<ProjectToBuild Include="$(ASCDir)module\ASC.Api\ASC.Api.Sample\ASC.Api.Sample.csproj"/>
```
and run the build\Build.bat file

**7.** IMPORTANT!!! Add ASC.Api.Sample.SampleApi to web\studio\ASC.Web.Studio\web.autofac.config
```
<component
          type="ASC.Api.Sample.SampleApi, ASC.Api.Sample"
          service="ASC.Api.Interfaces.IApiEntryPoint, ASC.Api"
          name="sample"/>
```

**8.** Build project, run website and test the method by making a query with jQuery
```
$.ajax({
    type: "POST",
    url: "http://localhost:port/api/2.0/sample/create.json",
    data: {value: "create"}
});
```