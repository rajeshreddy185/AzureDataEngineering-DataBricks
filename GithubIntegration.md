#### Azure Databricks Workspace to a GitHub repository



**Overview** 
This document outlines the standard procedure for connecting an Azure Databricks Workspace to a GitHub repository. 
It allows Data Engineers to use version control, collaborate on code, and implement CI/CD pipelines, moving away 
from manual notebook management.

2. Integration Methods
There are two primary ways to authenticate Databricks with GitHub.

Method A (The App): Uses the "Databricks GitHub App." Easier for organizations.

Method B (The Token - Recommended): Uses a Personal Access Token (PAT). 
                                    More robust and less likely to fail due to UI/redirect issues.


**Step-by-Step Configuration** 

**Step 1**: Generate a GitHub Personal Access Token (PAT)Log in to GitHub.
        Go to Settings (Click profile picture to Settings).
        Scroll to the bottom left sidebar to Developer settings.
        Click Personal access tokens to Tokens .
        Click Generate new token to Generate new token . 
**Note**: Enter "DatabricksIntegration" as name.
          Expiration: Set to X number of days (or custom).
          Select Scopes 
                - (Permissions):[x] repo (Full control of private repositories) or  whatever required
                – Critical[x] workflow (Update GitHub Action workflows) 
          Click Generate token.
          Copy the token immediately. (You will not see it again).

**Step 2**: Configure DatabricksLog in to your Azure Databricks Workspace.
            Click your Username/Profile Circle (Top Right) to Settings.
            Go to the Linked Accounts tab.Locate Git Integration.
            Git provider: Select GitHub from the dropdown.
            Token: Paste the PAT you copied in Step 
            Git username: Enter your GitHub username.
            Click Save.

**Step 3**: Clone a Repository (The "Repos" Feature)Go to the Workspace sidebar to Workspace to Users to Your Name.
            Click the Add button (Top Right) to Git Folder.
            Git repository URL: Paste your GitHub repo URL (e.g., https://github.com/rajeshreddy185/my-data-pipeline.git).
            Git provider: Select GitHub.
            Click Create Git Folder.



#### Troubleshooting Guide: Common Errors & Scenarios

The following scenarios address specific errors encountered during the setup process.

**Scenario 1**: 
    The "Sparse Checkout" LockError Message:"Sparse checkout mode cannot be updated for existing repositories. 
    Create a new repository if you want to update sparse checkout mode."
    Context:You tried to change settings (check/uncheck "Sparse checkout") on a folder that was already cloned. 
    Databricks locks the Git configuration to prevent corruption.
**Solution**:
    Do NOT try to fix the existing folder settings.Delete the folder: 
    Right-click the Repo folder to Move to Trash.
    Re-clone: Click Add to Git Folder.
    Decision Time:Option A (Full Repo): Leave "Sparse checkout mode" UNCHECKED.
    Option B (Partial Repo): CHECK it and immediately specify the folder path (e.g., src/pipelines).
    Click Create.


**Scenario 2**: 
    The "Link Access" DenialError Message:"The link to your GitHub account does not have access. 
    An admin of the repository must go here and install the Databricks GitHub app..."Context:Databricks tried to use the
    "GitHub App" method (Method A) but found the app wasn't installed or authorized for the specific repository you are
    trying to push to.
**Solution**:
    Go to GitHub Apps Settings: https://github.com/apps/databricks (or) https://github.com/settings/installationsFind 
    Databricks in the list.
    Click Configure.
    Under Repository access:Switch from "Only select repositories" to "All repositories".OR 
    Manually select the specific repo you are working on.
    Click Save.
    Retry the push in Databricks.

