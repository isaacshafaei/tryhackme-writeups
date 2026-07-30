#Security Footage
in this challenge we have a wireshark file pcap and we have to extract the recover footage from a camera:

after found out the this header of tcp and http is jpeg we decide use foremost tools like below to extract the images:
`foremost -i name.pcap -o out`
then we go to the output folder and find the flag
