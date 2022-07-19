[ODF External Mode CLI install Step-by-Step]

[Last tested OCP 4.10.22, ODF 4.10.5, RHCS 5.1z2]

Developed from [
[https://access.redhat.com/articles/5683981](https://www.google.com/url?q=https://access.redhat.com/articles/5683981&sa=D&source=editors&ust=1658254333403221&usg=AOvVaw3ArLNvIBo_mFP0-55ljSgo){.c14}
]{ 



1.   Create the openshift-storage namespace\
    [oc create -f ODF-EXT/odf-namespace.yaml]
2.   Create the openshift-storage-operatorgroup for the Operator\
    [oc create -f ODF-EXT/odf-operatorgroup.yaml]
3.   Subscribe to the odf-operator\
    [oc create -f ODF-EXT/odf-sub.yaml]
4.   Enable console plugin\
    \
    [oc patch console.operator cluster -n openshift-storage \--type json
    -p \'\[{\"op\": \"add\", \"path\": \"/spec/plugins\", \"value\":
    \[\"odf-console\"\]}\]\']



5.  [Create external cluster secret]

  1.  [The only location for the python script next referenced is in the
    ODF GUI on OCP. ]

    1.  [Under Installed Operators click on Openshift Data foundation]
    2.  [Choose Create StorageSystem]
    3.  [Select Connect an external storage platform and click Next]
    4.  [Click on the linked text Download script]

  2.   Run script as root on RHCS cluster, choose an appropriate pool
    name, note that you need to create the RBD pool in advance of
    running the script (do not forget to set application \"rbd\" on the
    RBD pool or it will warn). You will also need to know an RGW
    endpoint.\
    \
    [sudo python3 ceph-external-cluster-details-exporter.py
    \--rbd-data-pool-name test2_rbd \--rgw-endpoint 192.168.100.202:80
    -o external_cluster_details]



  3.  Put script output into file named [external_cluster_details on OCP
    admin node]
4.  Create [rook-ceph-external-cluster-details]{.c5}  secret from file\
    [oc create secret generic rook-ceph-external-cluster-details
    \--from-file=ODF-EXT/external_cluster_details -n
    openshift-storage]{.c5}

6.   Create Storage System and Storage cluster\
    [oc create -f ODF-EXT/odf-storage-system.yaml]
7.   Set default storage class (optional if you already have a default
    storage class set)\
    \
    [oc patch storageclass ocs-external-storagecluster-ceph-rbd -p
    \'{\"metadata\": {\"annotations\":
    {\"storageclass.kubernetes.io/is-default-class\":
    \"true\"}}}\']



8.  [Verify ]




