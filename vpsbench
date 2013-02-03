#!/bin/bash

#==============================================================================
# Copyright (c) 2013 Mario Gutierrez <mario@mgutz.com>
#
# Licensed under The MIT License
#==============================================================================


# Quick benchmark, always run
function bench_quick {
    local test_file=vpsbench__$$
    local tar_file=tarfile
    local now=$(date +"%m/%d/%Y")

    local cname=$( awk -F: '/model name/ {name=$2} END {print name}' /proc/cpuinfo )
    local cores=$( awk -F: '/model name/ {core++} END {print core}' /proc/cpuinfo )
    local freq=$( awk -F: ' /cpu MHz/ {freq=$2} END {print freq}' /proc/cpuinfo )
    local tram=$( free -m | awk 'NR==2 {print $2}' )
    local swap=$( free -m | awk 'NR==4 {print $2}' )
    local up=$(uptime|awk '{ $1=$2=$(NF-6)=$(NF-5)=$(NF-4)=$(NF-3)=$(NF-2)=$(NF-1)=$NF=""; print }')

    echo -n "Benching I/O ... "
    local io=$( ( dd if=/dev/zero of=$test_file bs=64k count=16k conv=fdatasync && rm -f $test_file ) 2>&1 | awk -F, '{io=$NF} END { print io}' )
    echo OK

    echo -n "Benching CPU. Bzipping 25MB file ... "
    dd if=/dev/urandom of=$tar_file bs=1024 count=25000 >>/dev/null 2>&1
    local tf=$( (/usr/bin/time -f "%es" tar cfj $tar_file.bz2 $tar_file) 2>&1 )
    rm -f tarfile*
    echo OK

    echo -n "Benching inbound network. Downloading 100MB file ... "
    local dl=$( _dl http://cachefly.cachefly.net/100mb.test )
    echo OK

    cat <<INFO
Share at https://github.com/mgutz/vpsbench/wiki/VPS-Hosts


_$now - VMPLAN - DATACENTER - OS - AUTHOR_
\`\`\`
CPU model: $cname
Number of cores: $cores
CPU frequency: $freq MHz
Total amount of RAM: $tram MB
Total amount of swap: $swap MB
System uptime: $up
I/O speed: $io
Bzip 25MB: $tf
Download 100MB file: $dl
\`\`\`


INFO

}

function _dl {
    wget -O /dev/null "$1" 2>&1 | awk '/\/dev\/null/ {speed=$3 $4} END {gsub(/\(|\)/,"",speed); print speed}'
}


# Bench a single download file.
function bench_dl {
    echo -n "Downloading from $1 ... "
    _dl "$2"
}


# Bench downloads from various places in the world.
function bench_downloads {
    echo

    bench_dl "CacheFly" \
        http://cachefly.cachefly.net/100mb.test

    bench_dl "Linode - Atlanta GA" \
        http://atlanta1.linode.com/100MB-atlanta.bin

    bench_dl "Linode - Dallas TX" \
        http://dallas1.linode.com/100MB-dallas.bin

    bench_dl "Linode - Tokyo JP" \
        http://tokyo1.linode.com/100MB-tokyo.bin

    bench_dl "Linode - London UK" \
        http://speedtest.london.linode.com/100MB-london.bin

    bench_dl "Leaseweb - Haarlem NL" \
        http://mirror.leaseweb.com/speedtest/100mb.bin

    bench_dl "Softlayer - Singapore" \
        http://speedtest.sng01.softlayer.com/downloads/test100.zip

    bench_dl "Softlayer - Seattle WA" \
        http://speedtest.sea01.softlayer.com/downloads/test100.zip

    bench_dl "Softlayer - San Jose CA" \
        http://speedtest.sjc01.softlayer.com/downloads/test100.zip

    bench_dl "Softlayer - Washington DC" \
        http://speedtest.wdc01.softlayer.com/downloads/test100.zip
}


# Ensure unix bench is installed (must be root)
function ensure_unixbench {
    test -d UnixBench && return

    apt-get install build-essential
    #apt-get install build-essential libx11-dev libgl1-mesa-dev libxext-dev

    local file_ver=UnixBench5.1.3
    wget -N http://byte-unixbench.googlecode.com/files/$file_ver.tgz
    tar xvfz $file_ver.tgz
}


# Perform comprehensive benchmark of cpu and disk
function bench_unixbench {
    ensure_unixbench

    pushd UnixBench > /dev/null
    # dont need graphics
    ./Run index
    popd > /dev/null
}


# Creates a 100MB file for download testing.
function create_big_file {
    echo -n Creating big file ...
    dd if=/dev/urandom of=bigfile bs=1024 count=102400
    echo OK
}


# Remove temporary files and directories used/created during benchmarking.
function remove_temporary_files {
    echo -n Removing temporary files ...
    rm -f vpsbench__*
    rm -f bigfile
    rm -rf UnixBench*
    rm -rf tarfile*
    echo OK
}


# Display usage.
function usage {
    cat <<USAGE
Usage: vpsbench [OPTION...]

-a Bench all
-d Bench downloads
-f Create 100M bigfile
-x Remove temporary files
-u Bench unixbench
USAGE
}


# The main script.
function main {
    bench_quick

    test $do_downloads && bench_downloads
    test $do_unixbench && bench_unixbench
}


# Parse command line options
while getopts :adfxu opt; do
    case "$opt" in
        a)
            do_downloads=true
            do_unixbench=true
            ;;
        d)
            do_downloads=true
            ;;
        f)
            create_big_file
            exit
            ;;
        x)
            remove_temporary_files
            exit
            ;;
        u)
            do_unixbench=true
            ;;
        ?)
            usage
            exit 2
            ;;
    esac
done

main

