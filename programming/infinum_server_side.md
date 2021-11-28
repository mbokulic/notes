heroku 12 factor app
use config outside code, always
git-secret tool
some tools: amazon key service, confidant, keywhiz
vault https://www.vaultproject.io
infinum created a cli for vault on github

**advanced sql functions**
over(partition by) splits the table into windows
...partition by order by ... rows between... changes the frames inside windows
over functions are expensive so use materialized views, you need to refresh these
not on mysql

aggregate to arrays if you need to filter across many boolean vars
only postgres