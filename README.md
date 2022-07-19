Developed from <https://access.redhat.com/articles/5683981>

1.  Create the openshift-storage namespace

    > oc create -f YAML/odf-namespace.yaml

2.  Create the openshift-storage-operatorgroup for the Operator

    > oc create -f YAML/odf-operatorgroup.yaml

3.  Subscribe to the odf-operator

    > oc create -f YAML/odf-sub.yaml

4.  Enable console plugin

    > oc patch console.operator cluster -n openshift-storage --type
    json -p \'\[{\"op\": \"add\", \"path\": \"/spec/plugins\",
    \"value\": \[\"odf-console\"\]}\]\'

5.  Create external cluster secret

    a.  The only location for the python script next referenced is in
        the ODF GUI on OCP.
        
        i.  Under Installed Operators click on Openshift Data foundation

        ii. Choose Create StorageSystem

        iii. Select Connect an external storage platform and click Next

        iv. Click on the linked text Download script

    b.  Run script as root on RHCS cluster, choose an appropriate pool
        name, note that you need to create the RBD pool in advance of
        running the script (do not forget to set application \"rbd\"
        on the RBD pool or it will warn). You will also need to know
        an RGW endpoint.

        sudo python3 ceph-external-cluster-details-exporter.py
        --rbd-data-pool-name test2_rbd --rgw-endpoint
        192.168.100.202:80 -o external_cluster_details

    c.  Put script output into file named external_cluster_details on
        OCP admin node

    d.  Create rook-ceph-external-cluster-details secret from file
    
        oc create secret generic rook-ceph-external-cluster-details
        --from-file=external_cluster_details -n
        openshift-storage

6.  Create Storage System and Storage cluster

    > oc create -f YAML/odf-storage-system.yaml

7.  Set default storage class (optional if you already have a default
    storage class set)

    > oc patch storageclass ocs-external-storagecluster-ceph-rbd -p
    > \'{\"metadata\": {\"annotations\":
    > {\"storageclass.kubernetes.io/is-default-class\": \"true\"}}}\'

8.  Verify
