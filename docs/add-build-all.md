Add build all script to build all Docker image


```
#!/bin/bash

set -e

DIMGS=`find . -type f -name 'Dockerfile'`
IMG_PREFIX=pilgrim2go/ddtraining
CPWD=$PWD

for DPATH in ${DIMGS[@]}; do
    DIMG=`dirname $DPATH | cut -c 3-`
    echo $DIMG
    cd $DIMG
    docker build -t $IMG_PREFIX/$DIMG . 
    cd $CPWD
done

```