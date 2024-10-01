1. Copy .devcontainer/devcontainer.env.example as .devcontainer/devcontainer.env and populate the secret values or ignore the ones not used by your dbt profiles (here we use key pair auth to Snowflake)

2. Change the docker image to your own in the devcontainer.json, or even modify the file to build directly from the repository. I have included the image we deploy at HomeToGo as an example, but we store it in harbor, and the building is living in another repository.

3. copy your DBT models in