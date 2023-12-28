Certainly! `tcpdump` is a powerful command-line packet analyzer that allows you to capture and analyze network traffic on a Unix-like operating system. Here's a cheat sheet that can help you get started with `tcpdump`:

### Basic Syntax:
```bash
tcpdump [options] [filter expression]
```

### Common Options:
- `-i`: Specify the network interface (e.g., eth0, wlan0).
- `-n`: Don't resolve hostnames.
- `-nn`: Don't resolve hostnames or port numbers.
- `-c`: Capture a specific number of packets and then exit.
- `-w`: Write the raw packets to a file for later analysis.
- `-r`: Read packets from a file (useful for offline analysis).

### Filtering:
- `host <ip>`: Capture packets to or from a specific IP address.
- `net <subnet>`: Capture packets to or from a specific subnet.
- `port <port>`: Capture packets with a specific port number.
- `src <ip>`: Capture packets with a specific source IP address.
- `dst <ip>`: Capture packets with a specific destination IP address.
- `tcp` or `udp`: Capture TCP or UDP packets.
- `icmp`: Capture ICMP packets.

### Examples:
1. **Capture all traffic on a specific interface:**
    ```bash
    tcpdump -i eth0
    ```

2. **Capture and display a specific number of packets:**
    ```bash
    tcpdump -c 10
    ```

3. **Capture packets to or from a specific IP address:**
    ```bash
    tcpdump host 192.168.1.100
    ```

4. **Capture packets on a specific port:**
    ```bash
    tcpdump port 80
    ```

5. **Capture TCP traffic on a specific interface and port:**
    ```bash
    tcpdump -i eth0 tcp port 22
    ```

6. **Capture packets in a specific subnet:**
    ```bash
    tcpdump net 192.168.1.0/24
    ```

7. **Capture ICMP packets:**
    ```bash
    tcpdump icmp
    ```

8. **Write packets to a file for later analysis:**
    ```bash
    tcpdump -w output.pcap
    ```

9. **Read packets from a file:**
    ```bash
    tcpdump -r input.pcap
    ```

### Advanced Filtering (BPF):
- BPF (Berkeley Packet Filter) expressions can be used for more complex filtering.

### Tips:
- Use `sudo` to run `tcpdump` with administrative privileges for better capture capabilities.
- Save the captured data to a file (`-w`) for detailed analysis with tools like Wireshark.

Remember that `tcpdump` offers extensive filtering options, so refer to the manual (`man tcpdump`) for more details on the available options and expressions.

### Examples

1. **Capture DNS Queries:**
    ```bash
    tcpdump -i eth0 port 53
    ```

2. **Capture HTTP Traffic:**
    ```bash
    tcpdump -i eth0 port 80 or port 8080
    ```

3. **Capture HTTPS Traffic (Port 443):**
    ```bash
    tcpdump -i eth0 port 443
    ```

4. **Capture SMTP Traffic (Port 25):**
    ```bash
    tcpdump -i eth0 port 25
    ```

5. **Capture FTP Traffic (Port 21):**
    ```bash
    tcpdump -i eth0 port 21
    ```

6. **Capture ICMP Echo Requests (Ping):**
    ```bash
    tcpdump -i eth0 icmp
    ```

7. **Capture Telnet Traffic (Port 23):**
    ```bash
    tcpdump -i eth0 port 23
    ```

8. **Capture ARP Requests:**
    ```bash
    tcpdump -i eth0 arp
    ```

9. **Capture Only IPv6 Traffic:**
    ```bash
    tcpdump -i eth0 ip6
    ```

10. **Capture Non-HTTP Traffic to a Specific IP:**
    ```bash
    tcpdump -i eth0 not port 80 and host 192.168.1.100
    ```

11. **Capture FTP Data (Passive Mode):**
    ```bash
    tcpdump -i eth0 port 20 or (port 21 and \(tcp[13] & 8 != 0\))
    ```

12. **Capture Traffic Between Two Hosts:**
    ```bash
    tcpdump -i eth0 host 192.168.1.100 and host 192.168.1.200
    ```

13. **Capture Traffic Based on Packet Size:**
    ```bash
    tcpdump -i eth0 greater 1000  # Capture packets larger than 1000 bytes
    ```

14. **Capture DNS Queries for a Specific Domain:**
    ```bash
    tcpdump -i eth0 port 53 and host example.com
    ```

15. **Capture SYN Packets (TCP Handshake):**
    ```bash
    tcpdump -i eth0 'tcp[13] & 2 != 0'
    ```

16. **Capture ECMP Echo Request (Ping)**

    ```bash
    sudo tcpdump -i eth0 icmp[icmptype] == 8
    ```

17. **Capture only ICMP Echo Reply**

    ```bash
    sudo tcpdump -i eth0 icmp[icmptype] == 0
    ```
