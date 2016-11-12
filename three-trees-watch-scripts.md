<img width="1339" alt="advanced-git-workshop_key" src="https://cloud.githubusercontent.com/assets/7910250/20234033/2319c308-a82b-11e6-829f-161c54f72a88.png">

<img width="1453" alt="3__kuychaco_c02qp3hqfvh8____src_three-trees-of-git__zsh__and_2__hub_log_--graph__--abbrev-commit__git__and_new_file" src="https://cloud.githubusercontent.com/assets/7910250/20234004/c9601f42-a82a-11e6-9dcb-9357cdcf9652.png">

Reference: [https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified#The-Workflow](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified#The-Workflow)

Set up watcher for working directory tree - `watch -n 1 -t show-working-dir`
```
#!/bin/bash

set -e

echo "Working Dir"
echo "sandbox"
echo ""
for file in $(ls); do
  echo " $file : $(cat $file)"
done
```

Set up watcher for index tree - `watch -n 1 -t show-stage`
```
#!/bin/bash

set -e

echo "Stage/Index"
echo "next commit snapshot"
echo ""
while read -r line; do
  filename=$(echo "$line" | cut -d '	' -f 2)
  sha=$(echo "$line" | cut -d ' ' -f 2)
  content=$(git cat-file -p $sha)
  echo "$sha $filename : $content"
done <<< "$(git ls-files --abbrev -s)"
```

Set up watcher for head tree - `watch -n 1 -t show-head`

```
#!/bin/bash

set -e

echo "Head"
echo "last commit snapshot"
echo ""
while read -r line; do
  filename=$(echo "$line" | cut -d '	' -f 2)
  sha=$(echo "$line" | cut -d ' ' -f 3 | head -c 7)
  content=$(git cat-file -p $sha)
  echo "$sha $filename : $content"
done <<< "$(git cat-file -p head^{tree})"
```




