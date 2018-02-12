FROM colvert22/functest-core

ARG BRANCH=master
ARG OPENSTACK_TAG=stable/pike
ARG OS_FAULTS_TAG=0.1.16
ARG ONAP_TAG=master
ARG ONAP_SUITE_PATH=/var/opt/OpenECOMP_ETE

COPY thirdparty-requirements.txt thirdparty-requirements.txt
RUN mkdir -p $ONAP_SUITE_PATH

RUN apk --no-cache add --virtual .build-deps --update \
        python-dev build-base linux-headers libffi-dev \
        openssl-dev libjpeg-turbo-dev && \
    wget -q -O- https://git.openstack.org/cgit/openstack/requirements/plain/upper-constraints.txt?h=$OPENSTACK_TAG | \
        sed -E s/^tempest==+\(.*\)$/-e\ git+https:\\/\\/github.com\\/openstack\\/tempest@\\1#egg=tempest/ \
        > upper-constraints.txt && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        -e git+https://github.com/openstack/requirements@$OPENSTACK_TAG#egg=openstack_requirements && \
    git clone --depth 1 https://github.com/openstack/os-faults.git -b $OS_FAULTS_TAG /src/os-faults && \
    update-requirements -s --source /src/openstack-requirements /src/os-faults/ && \
    pip install --no-cache-dir --src /src -cupper-constraints.txt \
        -chttps://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH \
        tempest /src/os-faults && \
    git clone --depth 1 https://git.onap.org/testsuite -b $ONAP_TAG $ONAP_SUITE_PATH && \
    git clone --depth 1 https://git.onap.org/testsuite/properties -b $ONAP_TAG /share/config && \
    git clone --depth 1 https://git.onap.org/testsuite/python-testing-utils -b $ONAP_TAG /src/testing-utils \
        (cd /src/testing-utils && pip install -e .) && \
    apk del .build-deps
COPY testcases.yaml /usr/lib/python2.7/site-packages/functest/ci/testcases.yaml
COPY *.robot $ONAP_SUITE_PATH/robot/testsuites/
CMD ["run_tests", "-t", "all"]
