# Advanced Docker Operations

## Extending container privileges with capabilities

**--cap-drop** CAP_CHOWN removes permisson access from RUN command
**--cap-add** CAP_CHOWN adds permisson access from RUN command
**--cap-drop** ALL
**--device** for devices
**--privileged** Can acces everything

Rootless mode

## Setting Container Resource Limits

Containers are composed of **name spaces** and **control groups**.
Name spaces specify the resources that containers can see or access on a system.
Control groups specify how much of those resources containers can consume.

|**--cpus 0.00**|**--memory1G**|
|---------------|--------------|
|- Sets maximum processor time|- Sets maximum amount of memory|

|**--cpu-period**|**--cpu-quota**|
|----------------|---------------|
|- Time to wait until receiving processor time|- Max amount of processor time to use when processor time received|
