## cronをリストするawk文



```bash
hostname=$(hostname); awk -v host="$hostname" '{ if ($0 ~ /^#/) next; command=""; for (i=6; i<=NF; i++) command = command $i " "; command = "\"" command "\""; print host "\t" $1 "\t" $2 "\t" $3 "\t" $4 "\t" $5 "\t" command; }' mycron

```

