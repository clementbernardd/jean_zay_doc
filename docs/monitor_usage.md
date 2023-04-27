# Monitor usage

Here is a list of commands to keep track of resources (GPU hours, disk usage)

| Command | Description | 
| ------- | ---------- | 
| `idracct` | Show the number of GPU hours used by each user & for how many jobs | 
| `idr_quota_user` | Amount of storage used in each directory by the user | 
| `idr_quota_project` | Amount of storage used in each directory by all users | 
| `idrquota -m` | Occupation of HOME directory (for user) | 
| `idrquota -s` | Occupation of STORE directory (for project) | 
| `idrquota -w` | Occupation of WORK directory (for project) | 
