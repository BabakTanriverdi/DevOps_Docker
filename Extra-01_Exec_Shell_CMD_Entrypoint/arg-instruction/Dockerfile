# FROM alpine
# ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-1/main/version.tar /version.tar
# RUN tar -xvf version.tar
# CMD ["cat", "/version/version"]

# FROM alpine
# ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-2/main/version.tar /version.tar
# RUN tar -xvf version.tar
# CMD ["cat", "/version/version"]

FROM alpine
ARG VERSION=1
ADD https://raw.githubusercontent.com/clarusway-aws-devops/version-${VERSION}/main/version.tar /version.tar
RUN tar -xvf version.tar
CMD ["cat", "/version/version"]