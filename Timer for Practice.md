## 🧱 OPTION 2 — Manual “stopwatch” with shell built-ins

Add a tiny timer function in your `~/.bashrc` or `~/.zshrc`:

`start_timer() { export CKAD_START=$(date +%s); } stop_timer() {   local end=$(date +%s)   local diff=$((end - CKAD_START))   echo "⏱️ Time spent: $((diff / 60)) min $((diff % 60)) sec" }`

Usage:

`start_timer # ⏳ do your question kubectl apply -f pod.yaml kubectl logs pod stop_timer`

✅ Great for full questions (5–8 min range).  
You can repeat this for each question and note the results in a text file.

## 📒 OPTION 3 — Auto-log every question time

If you want a mini scoreboard:

`log_time() {   local task="$1"   local end=$(date +%s)   local diff=$((end - CKAD_START))   echo "$(date '+%H:%M:%S') | $task | $diff sec" >> ~/ckad_time_log.txt }`

Workflow:

`start_timer # work on Question 5 kubectl create deploy web --image=nginx stop_timer log_time "Q5: Deployment"`

Then view your daily performance:

`column -t ~/ckad_time_log.txt`

✅ You’ll quickly see which domains take longest.