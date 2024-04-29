# What is K8S
**K8S is a container archestration tool**
- Supports stateless apps
- Supports stateful apps
- Autoscales containerized apps
- Allows resource request levels
- Allows resource limits
- Is extensible, and have lot of plugins
- Is open source and portable, which avoids vendor lock in.
# K8S Concepts
**Imperative configuration**
Achieve desired state of the configuration by issueing commands.
The imperative method is used for quick fixes.
**Declarative configuration**
you define the desired state, and k8s takes cares of maintaining/changing the current state to match the desired state. 
## K8S Object model
Each item k8s manages is represented by an object, and you can view and change these objects state attributes and state.

K8s objects have two important elements, object spec and status.
Object spec is the desired state defined by user.
Object status is the current state provided by K8S.
## Declarative management
you define the desired state, and k8s takes cares of maintaining/changing the current state to match the desired state of the objects. 

## K8S Components
