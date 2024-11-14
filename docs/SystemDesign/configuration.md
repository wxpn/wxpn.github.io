# Configuration

Configuration is a set of parameters or constants that your system or application is going to use but instead of storing them in the application code, you would store them in a seemingly isolated file. Its usually written in JSOn format or YAML foramt. We usually name them "config.json" or "condfig.yaml". There are two types of configuration:

## static configuration 
This is configuration that is bundled with your application code. Changing a single value in the configuration file, then entire application needs to be redeployed. There are pros and cons to this method:
- During the code review process if there are mistakes in the configuration file. Then its deployed with rest of the application. If this change causes to break the application, that breakage may be caught in the tests you may have written or quaity assurance process. This means static configuration is much safer.
- Its also going to be slower as need to redeploy the entire application before the changes are visible.

## Dynamic Configuration
These are completely seperated from your applcation code which means that when you change your configuration, that would have immediate ramification on the application. 
- This means that the configuration needs to be backed up by a database, and when application detects changes, these changes are incorporated immediately or quickly. There is lot of power and flexibility. 
- There is a lot of risk to this, as you dont have time to test the impact of the change.
	- The big companies usually use dynamic configuration however, they build safeguards to prevent damage, such as building a review system to validate the changes made, access controls such that only valid users can make changes and configuration system was deployed every x amount of time. Also, companies can implements this configuration change such that it would impact only a fraction of users to begin with, that way impact is limited and so more safely role out changes.

[back](../SystemDesign.md)