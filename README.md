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

- To free ram memory
```console
su -c "echo 3 >'/proc/sys/vm/drop_caches' && swapoff -a && swapon -a && printf '\n%s\n' 'Ram-cache and Swap Cleared'" root 
```

- To check ram memory
```console
free -h 
```