# find all pods and get the pods which are used highest CPU
- `kubectl top pod -A | sort --reverse --key 3 --numeric`
- `kubectl top pod --all-namespace` - Prints the top information of all pods in all namespaces. Other filters may also be applied.
- `sort --reverse --key 3 --numeric` - Sorts the input according to the numeric values found in column 3 in descending order.
- `head -10` - Prints only the top 10 lines.
