# install python without sudo

restart install from local if needed

```bash
rm -rf $HOME/local/python3.11
```


```bash
mkdir -p $HOME/local
cd $HOME/local
# https://www.openssl.org/source/
wget https://github.com/openssl/openssl/releases/download/openssl-3.5.2/openssl-3.5.2.tar.gz
tar -xvf openssl-3.5.2.tar.gz
cd openssl-3.5.2
./config --prefix=$HOME/local/openssl --openssldir=$HOME/local/openssl
make -j$(nproc)
make install
```

```bash
cd $HOME/local
# https://sourceware.org/libffi/
wget https://github.com/libffi/libffi/releases/download/v3.4.5/libffi-3.4.5.tar.gz
tar -xvf libffi-3.4.5.tar.gz
cd libffi-3.4.5
./configure --prefix=$HOME/local/libffi
make -j$(nproc)
make install
```

```bash
cd $HOME/local
# https://github.com/madler/zlib/releases
wget https://github.com/madler/zlib/releases/download/v1.3.1/zlib-1.3.1.tar.gz
tar -xvf zlib-1.3.1.tar.gz
cd zlib-1.3.1
./configure --prefix=$HOME/local/zlib
make -j$(nproc)
make install
```

```bash
cd $HOME
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
tar -xvf Python-3.11.0.tgz
cd Python-3.11.0

./configure \
    --prefix=$HOME/local/python3.11 \
    --with-openssl=$HOME/local/openssl \
    --enable-optimizations \
    CPPFLAGS="-I$HOME/local/openssl/include -I$HOME/local/libffi/include -I$HOME/local/zlib/include" \
    LDFLAGS="-L$HOME/local/openssl/lib -L$HOME/local/libffi/lib -L$HOME/local/zlib/lib" \
    LD_RUN_PATH="$HOME/local/openssl/lib:$HOME/local/libffi/lib:$HOME/local/zlib/lib"

make -j$(nproc)
make install
echo 'export PATH="$HOME/local/python3.11/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

if you don't need to pre-install openssl, libffi, zlib, you can simplify to:

```bash
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz; tar -xvf Python-3.11.0.tgz; cd Python-3.11.0
./configure --prefix=$HOME/local/python3.11 --enable-optimizations; make -j$(nproc); make install
echo 'export PATH="$HOME/local/python3.11/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

`--prefix=$HOME/local/python3.11` tells the build system to install everything there. After `make install`, the Python binaries, libraries, and headers are copied into that directory.

You can safely delete the tarball (Python-3.11.0.tgz) and the extracted source folder (Python-3.11.0) after make install completes. The installed Python does not depend on the source folder—it’s fully installed in `$HOME/local/python3.11`.

Your one-liner `echo 'export PATH=…' >> ~/.bashrc && source ~/.bashrc` ensures your shell sees this new Python immediately.



```bash
python3 --version
python3 -m venv venv
source venv/bin/activate
```

```bash
pip install --upgrade pip
pip install numpy
```

## ssl module not found

```bash
cd $HOME/local/openssl-3.5.2
make clean

# Configure OpenSSL with explicit shared library options
./configure linux-x86_64 \
    --prefix=$HOME/local/openssl \
    --openssldir=$HOME/local/openssl \
    shared \
    -fPIC \
    enable-shared \
    no-ssl3

make -j$(nproc)
make install

# Verify shared libraries were created
ls -la $HOME/local/openssl/lib/
```

```bash
cd $HOME
wget https://www.python.org/ftp/python/3.11.0/Python-3.11.0.tgz
tar -xvf Python-3.11.0.tgz
cd Python-3.11.0

# Set environment variables for the build
export PKG_CONFIG_PATH="$HOME/local/openssl/lib/pkgconfig:$HOME/local/libffi/lib/pkgconfig:$PKG_CONFIG_PATH"
export LD_LIBRARY_PATH="$HOME/local/openssl/lib:$HOME/local/libffi/lib:$HOME/local/zlib/lib:$LD_LIBRARY_PATH"

./configure \
    --prefix=$HOME/local/python3.11 \
    --with-openssl=$HOME/local/openssl \
    --enable-optimizations \
    --enable-shared \
    CPPFLAGS="-I$HOME/local/openssl/include -I$HOME/local/libffi/include -I$HOME/local/zlib/include" \
    LDFLAGS="-L$HOME/local/openssl/lib -L$HOME/local/libffi/lib -L$HOME/local/zlib/lib -Wl,-rpath,$HOME/local/openssl/lib -Wl,-rpath,$HOME/local/libffi/lib -Wl,-rpath,$HOME/local/zlib/lib" \
    LIBS="-lssl -lcrypto"

make -j$(nproc)
make install

# Add both PATH and LD_LIBRARY_PATH to bashrc
echo 'export PATH="$HOME/local/python3.11/bin:$PATH"' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH="$HOME/local/openssl/lib:$HOME/local/libffi/lib:$HOME/local/zlib/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
source ~/.bashrc
```

Perfect! The OpenSSL shared libraries are there in lib64. Now let's create the symlink and rebuild Python with the correct paths:
bash# Create a symlink from lib to lib64

```bash
cd $HOME/local/openssl
ln -sf lib64 lib

# Verify the symlink works
ls -la $HOME/local/openssl/lib/*.so*
```

```bash
cd $HOME/Python-3.11.0
make clean

export PKG_CONFIG_PATH="$HOME/local/openssl/lib64/pkgconfig:$HOME/local/libffi/lib/pkgconfig:$PKG_CONFIG_PATH"
export LD_LIBRARY_PATH="$HOME/local/openssl/lib64:$HOME/local/libffi/lib:$HOME/local/zlib/lib:$LD_LIBRARY_PATH"

./configure \
    --prefix=$HOME/local/python3.11 \
    --with-openssl=$HOME/local/openssl \
    --enable-optimizations \
    --enable-shared \
    CPPFLAGS="-I$HOME/local/openssl/include -I$HOME/local/libffi/include -I$HOME/local/zlib/include" \
    LDFLAGS="-L$HOME/local/python3.11/lib -L$HOME/local/openssl/lib64 -L$HOME/local/libffi/lib -L$HOME/local/zlib/lib \
             -Wl,-rpath,$HOME/local/python3.11/lib \
             -Wl,-rpath,$HOME/local/openssl/lib64 \
             -Wl,-rpath,$HOME/local/libffi/lib \
             -Wl,-rpath,$HOME/local/zlib/lib" \
    LIBS="-lssl -lcrypto"

make -j$(nproc) && make install
```

```bash
echo 'export LD_LIBRARY_PATH="$HOME/local/openssl/lib64:$HOME/local/libffi/lib:$HOME/local/zlib/lib:$LD_LIBRARY_PATH"' >> ~/.bashrc
source ~/.bashrc
```