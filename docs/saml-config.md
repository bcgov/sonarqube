# Configure SonarQube to integrate with BC Gov't Common Hosted Single Sign-on (CSS - AKA Redhat Single Single Sign-on. AKA - Platform Services Keycloak)

This SAML configuration procedure will allow IDIR users to log into your SonarQube instance. For this example we chose to have CSS manage the "Authorization" while continuing to control the "Authorization" through SonaeQube. We could have opted to have CSS manage the authorization through the use of roles. We are however, not able to use a group in the Azure AD to manage the access since IDIM does not return group information for each IDIR account. In practice this means each user who wishes to use SonarQube will first need to attempt a log into SonarQube. This will then add their name into the SonarQube system allowing a SonarQube administrator to grant them the permissions required to access the various SonarQube Projects.

## Common Hosted Single Sign-on (CSS)
 - request a new SAML SSO integration (https://bcgov.github.io/sso-requests)
 - config defaults except for:
    - Choose Identity Providers: Azure IDIR
    - Only need "Development" Environment
    - Redirect URIs: https://projectname-{licenseplate}-{namespace}.apps.silver.devops.gov.bc.ca/*  (We host our Sonarqube in the Tools namespace - config as appropriate)
        eg: https://sonarqube-abc123-tools.apps.silver.devops.gov.bc.ca/
 - Once the SSO integration has completed building you'll need to download the secrets. Navigation to CSS then choose your project. At the bottom of the page is "Integration Details". Choose to download the JSON file from the development (Azure IDIR). This file contains secrets you'll need for the SAML integration on SonarQube.

## SonarQube Config
We need to be sure to set the ServerBase for our domain. Navigate to:
    Administration -> Configuration -> General ->  Server base URL: https://{projectname}-{licenseplate}-{namespace).apps.silver.devops.gov.bc.ca/
    eg: https://sonarqube-123abc-tools.apps.silver.devops.gov.bc.ca/

Next we config the SAML within SonarQube
    Administration -> Configuration -> Authentication -> SAML
    The mappings (some of these you'll need from the JSON file previously download)
    - Application ID => Service Provider Entity ID
    - Provider Name => SAML (arbitrary name)
    - Provider ID => use Single Sign-on Service URL but strip out the /protocol/saml from the end. eg: https://dev.loginproxy.gov.bc.ca/auth/realms/standard
    - SAML login URL => Single Sign-on Service URL (just copy and paste from JSON file) eg: https://dev.loginproxy.gov.bc.ca/auth/realms/standard/protocol/saml
    - Identity Provider Certificate => x.509 certificate obtained from the JSON file.  eg: MIICn{redacted}abc123
    - SAML User Login Attribute => idir_username  (see https://user-images.githubusercontent.com/56739669/198112312-d860960b-283c-4f52-b0bb-2911ac0d04fb.png for details)
    - SAML User Name Attribute => display_name

Pressing "test your configuration" should return a JSON file with several of your IDIR populated fields.

# Tricks to test:
    We had to place a "*" in the Redirect URIs in development on CSS as we were getting a URI error. This was helpful to debug. We removed it once things were working.

# Authorization
We made all our SonarQube projects private and explicitly set the default group to allow zero permissions as any IDIR user can log into SonarQube and will be automatically added to the default group. We then require each user to log into SonarQube which added them to the system after which we can promote them to a group with appropriate permissions. (eg: admin or developer).
