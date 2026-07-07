`event.code : "1"  and process.command_line : *pdf*`



`event.code : "1" and process.parent.name:"mshta.exe"`

for finding the ip and port i used this:
`"rundll32.exe" and winlog.event_id : 3`


for this question i used below command: fodhelper.exe
`process.name:"fodhelper.exe"`

i used github to find the the url:
`*github*`


for finding the user credential find i used:
`*sekurlsa*`


for findinf the enumaration file :
`process.command_line:*\\\\*`

for finding the below question:
After getting the contents of the remote file, the attacker used the new credentials to move laterally. What is the new set of credentials discovered by the attacker? (format: username:password) and the next question
`winlog.event_id : 1 AND *PSCredential*`
