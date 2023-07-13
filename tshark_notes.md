# tshark notes

## Misc
- List of all available `tshark` fields
```
tshark -G fields
- row starts with P: protocol
- row starts with F: fields in that protocol
```

- specify name resolution options
```
dns=" -N dnN"
```

## Run `tshark` on a pcap:
- Each line in the output contains relevant information at packet level
```
bash pcap2csv.sh sample_traffic.pcap > sample_traffic.csv
```
- Flow id / payload size and hash
```bash
#!/bin/bash

file="$1"
time_format=" -t e"
display=" -T fields"

# specify tshark output fields after -Tfields option
fields+=" -e _ws.col.Time"
fields+=" -e ip.src"
fields+=" -e ipv6.src"
fields+=" -e tcp.srcport"
fields+=" -e _ws.col.Protocol"
fields+=" -e ip.dst"
fields+=" -e ipv6.dst"
fields+=" -e tcp.dstport"
fields+=" -e tcp.len"
fields+=" -e tcp.payload"
fields+=" -e _ws.col.Info"

# specify tshark filter for packets
filter=""
filter+=" (((tcp.port==5222) || (tcp.port==443)) and (tcp.len > 0)) "

# specify file separator and quote
separator=" -E separator=|"
quotes=" -E quote=n" # if separator=, then must use double quote since info might have comma

echo "timestamp|flow|src_ip|src_port|proto|dst_ip|dst_port|tcp_len|tcp_payload|info"
# fields index in awk following tshark fields above (-e)
tshark -r "$file" $time_format $display $fields $decode $separator $quotes $filter | \
awk -F'|' '{
        timestamp = $1;
        src_ip = $2$3;
        src_port = $4;
        dst_ip = $6$7;
        dst_port = $8;
        proto = $5;
        tcp_len = $9;
        tcp_payload = $10;
        info = $11;
        flow = ((src_port==443)||(src_port==5222)) ? dst_ip":"dst_port"-"src_ip":"src_port : src_ip":"src_port"-"dst_ip":"dst_port;
        tcp_len = ((src_port==443)||(src_port==5222)) ? "-"tcp_len : tcp_len;
        print $1"|"flow"|"src_ip"|"src_port"|"proto"|"dst_ip"|"dst_port"|"tcp_len"|"substr(tcp_payload,1,64)"|"info;
       }'
```

- Packet Data Unit (PDU)
```bash
#!/bin/bash

file="$1"
time_format=" -t e"
display=" -T fields"

# General columns
fields=" -e _ws.col.Time"
fields+=" -e tls.handshake.extensions_server_name" # Server Name Indicater (SNI) is more accurate than DNS
fields+=" -e _ws.col.Source"
fields+=" -e _ws.col.Destination"
fields+=" -e _ws.col.Protocol"
fields+=" -e _ws.col.Length"

# TCP protocol
fields+=" -e tcp.stream"
fields+=" -e tcp.len"
fields+=" -e tcp.srcport"
fields+=" -e tcp.dstport"
fields+=" -e tls.record.length"
fields+=" -e tcp.ack"

# General column
fields+=" -e _ws.col.Info"

# specify tshark filter for packets
filter=""
filter+=" (tls.app_data) || (tls.handshake.type==1)" # only get the PDUs with TLS record, or the Client Hello for SNI

# specify file separator and quote
separator=" -E separator=|"
quotes=" -E quote=n" # if separator=, then must use double quote since info might have comma

echo "time|sni|src|dst|proto|pkt_len|tcp_stream|tcp_len|tcp_sport|tcp_dport|tls_len|tcp_ack|info"
tshark -r "$file" $dns $time_format $display $fields $separator $quotes $filter \
| sort -t'|' -k1,1
```

- RTP SSRC

[RTP](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol)
```bash
#!/bin/bash

file="$1"
time_format=" -t e"
display=" -T fields"

# specify tshark output fields after -Tfields option
fields+=" -e _ws.col.Time"
fields+=" -e frame.time_relative"
fields+=" -e ip.src"
fields+=" -e ipv6.src"
fields+=" -e tcp.srcport"
fields+=" -e udp.srcport"
fields+=" -e _ws.col.Protocol"
fields+=" -e ip.dst"
fields+=" -e ipv6.dst"
fields+=" -e tcp.dstport"
fields+=" -e udp.dstport"
fields+=" -e _ws.col.Info"
fields+=" -e rtcp.senderssrc"
fields+=" -e rtp.ssrc"
fields+=" -e rtp.payload" # only get the first 8 bytes is enough

# decode as RTP
decode=" -o rtp.heuristic_rtp:TRUE"

# specify tshark filter for packets
#filter=""
filter+=" ((tcp.len > 0) || (udp.length > 0)) "

# specify file separator and quote
separator=" -E separator=|"
quotes=" -E quote=n" # if separator=, then must use double quote since info might have comma

echo "time|time_relative|src_ip|src_port|proto|dst_ip|dst_port|info|ssrc|rtp_payload"
tshark -r "$file" $time_format $display $fields $decode $separator $quotes $filter | \
awk -F'|' '{print $1"|"$2"|"$3$4"|"$5$6"|"$7"|"$8$9"|"$10$11"|"$12"|"$13$14"|"substr($15,1,16)}'
```