## To update a deployment

- `kubectl apply -f foo-deployment.definition.yaml`

## Get the status of a rollout

- `kubectl rollout status deployment/foo-deployment` 

## To undo a change (rollback)

- `kubectl rollout undo deployment/foo-deployment`

