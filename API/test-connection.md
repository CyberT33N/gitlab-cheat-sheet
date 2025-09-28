


# Local 


Make sure to enable `GitHub` and `Repository by URL`:
- https://gitlab.local.com/admin/application_settings/general#js-import-export-settings

Test if access is granted:

Get the CA chain:
```shell
openssl s_client -showcerts -connect gitlab.local.com:443 -servername gitlab.local.com </dev/null \
| awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/{print}' > /tmp/gitlab-ca-chain.pem
```

```shell
curl -sS --cacert /tmp/gitlab-ca-chain.pem -H "PRIVATE-TOKEN: $GITLAB_TOKEN" https://gitlab.local.com/api/v4/application/settings | jq '{import_sources, allow_local_requests_from_web_hooks_and_services, allow_local_requests_from_system_hooks}' | cat
```
- Should be `"import_sources": ["github", "git"]`

Test if you can create a repository:
```shell
 TS=$(date +%s); curl -sS -i --cacert /tmp/gitlab-ca-chain.pem -H "PRIVATE-TOKEN: $GITLAB_TOKEN" -H "Content-Type: application/json" --data "{\"name\":\"import-db-test-$TS\",\"path\":\"import-db-test-$TS\",\"visibility\":\"private\",\"default_branch\":\"main\",\"import_url\":\"https://github.com/git/git.git\"}" https://gitlab.local.com/api/v4/projects | head -n 20 | cat
HTTP/2 201 
```

