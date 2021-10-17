
## Error: kubernetes - Couldn't able to join master node - error execution phase preflight: couldn't validate the identity of the API Server
**Res:** 
On a standalone ubuntu vm , disable the firewall and check  
```sh
ufw status
sudo ufw disable
```
On an aws ec2  
`Expose port:6443 in Security Group`
