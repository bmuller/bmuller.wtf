---
title: "Stunning: Determining Your Public IP"
date: 2014-05-17
summary: "Determining Your Public IP with Ruby via Google"
---
Programmatically fetching your public IP address (aka, internet visible IP) can be tough.  Most often, I've done something silly like fetching [whatismyip.com](http://whatismyip.com) and then parsing the page.  That introduces a pretty bad dependency.

Fortunately, there's actually a protocol for asking for your internet visible IP called [Session Traversal Utilities for NAT](https://en.wikipedia.org/wiki/STUN) (or STUN).  It's used by services that require peer-to-peer connection negotiation (like Skype, Google hangouts, etc).

Here's an example Ruby script that will hit up a few public STUN servers (starting with Google's) and return your found IP:

```ruby
require 'socket'
require 'timeout'

class StunClient
  def initialize(host, port)
    @host = host
    @port = port
  end

  def get_ip
    begin
      Timeout::timeout(0.5) {
        socket = UDPSocket.new
        data = [0x0001,0].pack("nn") + Random.new.bytes(16)
        socket.send(data, 0, @host, @port)
        data, _ = socket.recvfrom(1000)
        type, length = data.unpack("nn")

        # if not a message binding response
        return nil unless type == 0x0101

        data = data[20..-1]
        while data.size > 0
          type, length = data.unpack("nn")
          # if attr type is ATTR_MAPPED_ADDRESS, return it
          if type == 0x0001
            values = data[4...4+length].unpack("CCnCCCC")
            return values[3..-1]*"."
          end
          data = data[4+length..-1]
        end

        return nil
      }
    rescue Timeout::Error
      return nil
    end
  end

  def self.get_ip
    servers = [["stun.l.google.com", 19302], ["stun.ekiga.net", 3478], ["stunserver.org", 3478]]
    servers.each do |host, port|
      ip = StunClient.new(host, port).get_ip
      return ip unless ip.nil?
    end
    nil
  end
end


puts StunClient.get_ip
```