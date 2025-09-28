# Delete all
- https://docs.gitlab.com/api/projects/


## local

```shell
#!/bin/bash
set -euo pipefail

# Config (env overrides allowed)
GITLAB_URL=${GITLAB_URL:-https://gitlab.local.com}
CACERT=${CACERT:-/tmp/gitlab-ca-chain.pem}
: "${GITLAB_TOKEN?GITLAB_TOKEN is required}"

page=1
while :; do
  list=$(curl -sS --cacert "$CACERT" -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
         "$GITLAB_URL/api/v4/projects?owned=true&simple=true&include_pending_delete=true&per_page=100&page=$page")
  rows=$(printf '%s' "$list" | jq -r '.[] | [.id, .path_with_namespace] | @tsv')
  [ -z "$rows" ] && break
  printf '%s\n' "$rows" | while IFS=$'\t' read -r id path; do
    [ -z "$id" ] && continue
    path_enc=$(printf '%s' "$path" | jq -sRr @uri)
    echo "Marking for deletion: $path ($id)"
    curl -sS -X DELETE --cacert "$CACERT" -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
      "$GITLAB_URL/api/v4/projects/$id" >/dev/null || true
    echo "Permanently removing: $path ($id)"
    curl -sS -X DELETE --cacert "$CACERT" -H "PRIVATE-TOKEN: $GITLAB_TOKEN" \
      "$GITLAB_URL/api/v4/projects/$id?permanently_remove=true&full_path=$path_enc" >/dev/null || true
  done
  page=$((page+1))
done
``
