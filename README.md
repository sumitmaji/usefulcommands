# Useful Commands

- Stop the kubelet service running in vm
```console
systemctl stop kubelet
```

- Start the kubelet service running in vm
```console
systemctl start kubelet
```

- Check the status of kubelet service
```console
systemctl status kubelet
```

- Debugging logs in kubelet
```console
journalctl -u kubelet 
```