# Install Golang 1.20 on Ubuntu 22

Golang 1.20 is required for running Polygon Supernets v1.0.0 upwards.
## 1. Update system
```
sudo apt-get update
sudo apt-get -y upgrade
```

## 2. Download Go binary
Next we download the Go binary file. It is available [here](https://golang.org/dl/).
```
mkdir temp
cd temp ( to the folder you created in previous step)
wget https://dl.google.com/go/go1.20.linux-amd64.tar.gz
```
Extract the downloaded tar and installing into the desired location in the system. Golang's Documentation recommends installing it under `/usr/local/go`.
```
sudo tar -xvf go1.20.linux-amd64.tar.gz
sudo mv go /usr/local
```

## 3. Environment Setup
We need to set three Go language environment variables  -`GOROOT`, `GOPATH`, `PATH`
`GOROOT` - the path where Golang is installed in the mahchine
`GOPATH` - is the location of the working directory

In your `.bashrc` / `.zshrc` file add above global variables at the end of the file.
```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

## 4. Renewing the shell sessions
```
source ~/.profile
```
As a final check, run the command below.
```
go version
```
If Golang is installed correctly you should see the following output:
```
go version go1.20 linux/amd64
```