BASE := $(subst -, ,$(notdir ${CURDIR}))
ORG  := geomesa
REPO := $(word 2, ${BASE})-$(word 3, ${BASE})
IMG  := quay.io/${ORG}/${REPO}
DIST_TARBALL := tarballs/geomesa-accumulo-dist_2.11-${GEOMESA_VERSION}-bin.tar.gz
BUILD_TARBALL := ${HOME}/.m2/repository/org/locationtech/geomesa/geomesa-accumulo-dist_2.11/${GEOMESA_VERSION}/geomesa-accumulo-dist_2.11-${GEOMESA_VERSION}-bin.tar.gz
RUNTIME := geomesa-accumulo-distributed-runtime_2.11-${GEOMESA_VERSION}.jar

build: ${RUNTIME} dockerfile
	docker build \
                --build-arg GEOMESA_VERSION=${GEOMESA_VERSION} \
                --build-arg TAG=${TAG} \
                --build-arg ACCUMULO_VERSION=${ACCUMULO_VERSION} \
		-f target/Dockerfile \
                -t ${IMG}:${TAG} .

dockerfile:
	mkdir -p target
	sed 's/__TAG__/'"${TAG}"'/' Dockerfile.template > target/Dockerfile


${DIST_TARBALL}: ${BUILD_TARBALL}
	(mkdir -p tarballs; cd tarballs ;  cp ${BUILD_TARBALL} . )

${BUILD_TARBALL}:
	if [ ! -d $HOME/workspace ] ;then mkdir -p $HOME/workspace ;fi
	pushd $HOME/tmp/builds
	if [ ! -d $HOME/tmp/builds/geomesa ] ;then git clone http://github.com/locationtech/geomesa ;fi
	cd geomesa
	git fetch origin -v && git pull && git fetch upstream -v
	git checkout geomesa-${GEOMESA_VERSION}
	git pull --rebase
	build/mvn clean install -DskipTests
	popd

${RUNTIME}: ${DIST_TARBALL}
	tar zxvf $<
	cp geomesa-accumulo_2.11-${GEOMESA_VERSION}/dist/accumulo/${RUNTIME} .

publish: build
	docker push ${IMG}:${TAG}

clean:
	rm -rf geomesa-accumulo_2.11-*
	rm -rf tarballs

cleanest: clean
	rm -f geomesa-accumulo-dist*
	rm -rf tarballs
