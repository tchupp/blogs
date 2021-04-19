# kubectl favorites

### Count pods by image

```bash
kc get pods --all-namespaces -o jsonpath="{..image}" \
  | tr -s '[[:space:]]' '\n' \
  | sort \
  | uniq -c
```

### Group pods by node

```bash
kc get pods --all-namespaces -ojson \
  | jq '.items[] | {name: .metadata.name, node: .spec.nodeName}' \
  | jq -s 'group_by(.node)[] | { (.[0].node): [.[].name] }'
```
