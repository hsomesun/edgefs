SRCDIR=$(NEDGE_HOME)

ifdef NEDGE_NDEBUG
DEBUG_FLAGS=-DUSE_JE_MALLOC
DEBUG_LDFLAGS=-ljemalloc -fno-omit-frame-pointer
else
DEBUG_FLAGS=-fsanitize=address -fno-omit-frame-pointer -fno-common

ifeq (${CC},clang)
DEBUG_LDFLAGS=-fsanitize=address
else
DEBUG_LDFLAGS=-lasan
endif

endif

TARGET := grpc-s3x
SRCS := main.go $(wildcard */*.go)

.PHONY: all clean

.get:
	GOPATH=`pwd` go get -v || true
	GOPATH=`pwd` go get -u github.com/golang/protobuf || true
	GOPATH=`pwd` make -C src/github.com/golang/protobuf
	GOPATH=`pwd` go get -u google.golang.org/grpc || true
	GOPATH=`pwd` go get github.com/fullstorydev/grpcurl || true
	GOPATH=`pwd` go get github.com/pborman/getopt || true
	GOPATH=`pwd` go install github.com/fullstorydev/grpcurl/cmd/grpcurl || true
	./get-protobuf.sh
	touch $@

.proto: .get $(wildcard */*.proto)
	for d in status; do \
		./bin/protoc --proto_path=./$$d --plugin=./bin/protoc-gen-go --go_out=plugins=grpc:$$d ./$$d/$$d.proto; \
	done
	touch $@

$(TARGET): $(SRCS) .get .proto
	GOPATH=`pwd` CGO_LDFLAGS="-L$(SRCDIR)/lib -lccow -lccowutil -lnanomsg $(DEBUG_LDFLAGS)" \
		CGO_CFLAGS="$(DEBUG_FLAGS) -I$(SRCDIR)/include/ccow -I$(SRCDIR)/include" \
		go build -o grpc-s3x main.go

fmt:
	gofmt -e -s -w status main.go

clean:
	rm -rf $(TARGET) src .get .get bin pkg $(wildcard */*.pb.go)

install: $(TARGET)
	mkdir -p $(DESTDIR)$(SRCDIR)/sbin 2> /dev/null || true
	cp -a $(TARGET) $(DESTDIR)$(SRCDIR)/sbin
	mkdir -p $(DESTDIR)$(SRCDIR)/etc/ssl 2> /dev/null || true
	cp -a default.key $(DESTDIR)$(SRCDIR)/etc/ssl/ssl.key
	cp -a default.crt $(DESTDIR)$(SRCDIR)/etc/ssl/ssl.crt

all: install

uninstall:
	rm -f $(DESTDIR)$(SRCDIR)/sbin/$(TARGET)