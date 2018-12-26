# SIC43NT Tag authenticity verification using rolling code

This project provides an example ASP.NET Core project for SIC43NT rolling code authentication web application. SIC43NT tag provider can apply this concept to allow SIC43NT tag holders to verify 
their tags as well as communicate to them regarding to the tag status.

## Getting Started

### Installing on Microsoft Azure Web App 

#### Prerequisites

* SIC43NT Tag
* Android NFC Phone with [SIC43NT Writer](https://play.google.com/store/apps/details?id=com.sic.app.sic43nt.writer) App
* [Microsoft Azure Account](https://azure.microsoft.com/) 
* [Azure Command Line / Azure CLI](https://docs.microsoft.com/en-us/cli/azure) from [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview) in Azure Portal or locally [install](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) on your macOS, Linux or Window machine.

#### Step 1 : Create a resource group

Create a new resource group to contain this new sample web app by using Azure command line.
The following example creates a new resource group with name "sic43nt-sample-rg" and the location is in "West Europe". 

```
az group create --name sic43nt-sample-rg --location "West Europe"
```

#### Step 2 : Create a new Free App Service Plan
Create a new Free App Service Plan by using Azure command line. The following example creates a new Free App service plan with name "sic43nt_samplePlan" in resource group "sic43nt-sample-rg".

```
az appservice plan create --name sic43nt_samplePlan --resource-group sic43nt-sample-rg --sku FREE
```

#### Step 3 : Create a Web App 
Create a new Web App by using Azure command line. The following example creates a new Web App. Please replace '<app_name>' with a globally unique app name (valid characters are 'a-z', '0-9', and '-'). 

```
az webapp create --resource-group sic43nt-sample-rg --plan sic43nt_samplePlan --name <app_name>
```

#### Step 4 : Deploy the sample app using Git
Deploy source code from GitHub to Azure Web App using Azure command line. The following example deploy source code from https://github.com/SiliconCraft/sic43nt-server-aspnetcore.git in master branch to a Web App name '<app_name>' in resource group "sic43nt-sample-rg".
```
az webapp deployment source config --repo-url https://github.com/SiliconCraft/sic43nt-server-aspnetcore.git --branch master  --name <app_name> --resource-group sic43nt-sample-rg
```

#### Step 5 : Customize SIC43NT Tag
Use SIC43NT Writer App on Android NFC Phone to customize SIC43NT Tag as the explanation below.
* RLC MODE Tab (In case of default tag, this RLC mode can leave with default factory value)
  * **Rolling Code Mode**
    * Rolling Code keeps changing.
  * **Rolling Code Key**
    * FFFFFF + Tag UID (i.e. The Key of the Tag with UID = "39493000012345" is "FFFFFF39493000012345".)

* NDEF MESSAGE Tab
  * **MIME:** URL/URI
  * **Prefix:** https://
  * **NDEF Message:** <app_name>.azurewebsites.net/?d=
  * **Dynamic Data**
    * **UID:** Checked
    * **Tdata:** Checked
    * **Rolling Code:** Checked

After completely customize SIC43NT Tag with the setting above, each time you tap the SIC43NT tag to NFC Phone (iPhone, Android or any NDEF support device), the web page will display a table of Tamper Flag, Time Stamp value and Rolling Code value which keep changing. Especially for the rolling code value, it will be a match between "From Tag" and "From Server" column. This mean that server-side applicationm (which calculate rolling code based on same Rolling Code Key) can check the authenicity of SIC43NT Tag.

## Usage
To develop your own SIC43NT Web service, you can pick class **KeyStream** [KeyStream.cs](https://github.com/SiliconCraft/sic43nt-server-aspnetcore/blob/master/SIC43NT_Webserver/Utilities/KeyStream/KeyStream.cs) and class **Encrypt** [Encrypt.cs](https://github.com/SiliconCraft/sic43nt-server-aspnetcore/blob/master/SIC43NT_Webserver/Utilities/KeyStream/Encrypt.cs) to your own ASP.net project. These classes are utility for rolling code calculation. 

The method *stream* of KeyStream calculate rolling code. It requires *80 bits-Key* (input as a 20-characters hexadecimal string) and *32 bits Time Stamp* or *32 bits iv* (input as a 8-characters hexadecimal string).

The class which calling this method is **IndexModel** in [Index.cshtml.cs](https://github.com/SiliconCraft/sic43nt-server-aspnetcore/blob/master/SIC43NT_Webserver/Pages/Index.cshtml.cs) which is an example showing how to use it. The main content of  **IndexModel** relating to rolling code calculation is shown below.


```C#
public void OnGet(string d)
{
    //...
    uid = d.Substring(0, 14);
    flagTamperTag = d.Substring(14, 2);
    timeStampTag_str = d.Substring(16, 8);
    timeStampTag_uint = UInt32.Parse(timeStampTag_str, System.Globalization.NumberStyles.HexNumber);
    rollingCodeTag = d.Substring(24, 8);
    default_key = "FFFFFF" + uid;
    rollingCodeServer = KeyStream.stream(default_key, timeStampTag_str, 4);
    //...
    // Criteria to check authenicity of SIC43NT Tag.
    // Compare *rollingCodeServer* against *rollingCodeTag*
    // Confirm *timeStampTag_uint* of this UID is increasing ( more than previous value of timeStampTag_uint for this UID ).
}
```

## License

This project is licensed under the MIT License - see the [LICENSE.txt](LICENSE.txt) file for details

