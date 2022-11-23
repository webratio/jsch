# Fork of JSch-0.1.55 for Java 1.7

See original [README](README)

## Building

- Download the latest **Ant 1.9** from the [official site](https://ant.apache.org/bindownload.cgi) and place its `lib` directory into the `tools` directory (e.g. you should have `tools/lib/ant.jar`). *Do not use Ant 1.10 or newer* if you wish to compile with Java 1.7 or earlier.
- Set `JAVA_HOME` to a JDK 1.7 installation directory.

- Run `build.bat` (on Windows) or `build.sh` (on Linux).

The resulting JAR is placed in `dist/lib`.

## Changes in This Fork

### Support for rsa-sha2-256 and rsa-sha2-512

RSA keys may be negotiated with servers using new SHA-2 signature algorithms in addition to SHA-1, which has been removed from OpenSSH.

### Support for direct-streamlocal@openssh.com

Example: (see [SocketForwardingL.java](examples/SocketForwardingL.java))
```java
session.connect(30000);   // making a connection with timeout.

final int boundPort = session.setSocketForwardingL(null, 0, "/var/run/docker.sock", null, 1000);

URL myURL = new URL("http://localhost:" + boundPort + "/_ping");
HttpURLConnection myURLConnection = (HttpURLConnection) myURL.openConnection();
System.out.println("Docker Ping http response code (" + myURL + "): " + myURLConnection.getResponseCode());

session.disconnect();
```

or directly:

```java
final ChannelDirectStreamLocal channel = (ChannelDirectStreamLocal) session.openChannel("direct-streamlocal@openssh.com");
try {
    final OutputStream outputStream = channel.getOutputStream();
    final InputStream inputStream = channel.getInputStream();

    channel.setSocketPath("/var/run/docker.sock");
    channel.connect();

    String cmd = "GET /_ping HTTP/1.0\r\n\r\n";
    try (final PrintWriter printWriter = new PrintWriter(outputStream)) {
        printWriter.println(cmd);
        printWriter.flush();
    }
    try (BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
        for (String line; (line = reader.readLine()) != null; ) {
            System.out.println(line);
        }
    }
} catch (IOException exc) {
    exc.printStackTrace();
} finally {
    channel.disconnect();
}
```
