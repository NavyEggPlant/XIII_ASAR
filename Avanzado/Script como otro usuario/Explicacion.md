A recommended security practice nowadays is to use a less privileged account when logging into domain joined computers. The idea behind this is that if the session becomes compromised (such as from social attack like “you may already be a winner! emails) the compromise does not have the user’s more privileged credentials to do bad things with.
This challenge of minimizing the exposure to compromise yet still being able to administer and resolve an Active Directory domain environment is a tough one. Since the user is not logged on as a privileged one many of the AD specific tasks they may need to do will simply not work as expected.
An additional related scenario is service accounts which application servers are running as - an explicit security barrier.  As an administrator though it is common to need to check access or other settings as that service. Without a method to do so as that service identity there’s a lot of guesswork involved. A common workaround in the past was to log on interactively as that service identity which increase the security exposure of the service identity to all of the applications and services which run in an interactive session but do not in service sessions.
I’ve written this PowerShell script in order to overcome these scenarios and allow running PowerShell scripts as a specific identity. In Windows the identity is tied to a Windows Process. In order to start a script or other item as a different identity and not use the implicit process you are logged on as we need to create the process and explicitly tell Windows to use an alternate identity.  That’s what this script does, using the [System.Diagnostics.Process.Start](https://msdn.microsoft.com/en-us/library/system.diagnostics.process.start(v=vs.110).aspx) method.
The specific scenario that brought this script about was a need to test and view certificates for a service identity. Since certificate store objects can only be accessed and reliably tested as the identity that should have access to them we needed a way to run PowerShell scripts as that service identity. To that end I wrote this script to call one of my other scripts for certificate chaining (which is here on TechNet).

A few things about this script:

- The script takes one parameter: ScriptPath. This needs to be the full local path to the script along with script name.
- The script will not work against remote destinations (where script is on a share and not on local computer the PowerShell console is on).
- The script will display the results in the same PowerShell window the script was called from.
- The script will have the console wait until the called script completes before returning to the prompt.
- Any error messages received by the called script will be shown in the PowerShell console. This was a common pain point that I was happy to resolve for when I needed to troubleshoot the script processing or why the called script was failing.
- The script will prompt for the credentials to use.
- When complete a message of `Script $Scriptname has finished` will be shown.
- The script has been tested on Windows Server 2008 and later.
- If you need to do actions related to user registry, or certificates for the called identity, simply add the line below after line 41 but before line 52: `$RunScriptProcess.StartInfo.LoadUserProfile = $true`

Here’s an example of running the script:
```
PS C: \Scripts> .\startscriptasprocess.ps1 -ScriptPath "c:\scripts\isdomainadmin.ps1"
User CONTOSO\jimbob is not member of the built in Domain Admins group in domain contoso.com.Script .\isdomainadmin.ps1 has finished.
```
Hopefully this script helps make your life easier as you securely administer your environment.

From [Run Script As A Process ](https://gallery.technet.microsoft.com/scriptcenter/Run-Script-As-A-Process-53e0c56a) on Microsoft´s Technet
