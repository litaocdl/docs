Helm is the k8s package management tools. like `apt/yum` in k8s, solve the service dependency in k8s. 

K8s hook (Pod level)

  * initContainer
  * postStart
  * preStop

Helm hook: (pod level)

  * pre-install 
  * post-install 
  * pre-delete
  * post-delete
  * pre-upgrade 
  * post-upgrade 
  * pre-rollback
  * post-rollback 


### Commands

create a template
`helm create webserver` 

dry run a helm
`helm install --name webserver --dry-run --debug webserver/`

when one pod change in helm, use --set to change the value in `Values` file
`helm upgrade webserver --set image.tag=1.7 `

check the deploy history
`helm history  webserver`

```
REVISION UPDATE
1
2
3
```
check the history config revision=1
`helm get webserver --revision 1`
check the increment value change
`helm get values webserver --revision 1`

rollback
`helm rollback webserver 1`

helm is using k8s config map under `kube-system` to store the history. 

list helm repo, check chartmuseum
`helm repo list`
