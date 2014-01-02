hekad
=========

The hekad daemon is the core component of the heka project, which handles routing messages, generating metrics, aggregating statsd-type messages, running plugins on the messages, and sending messages to the configured destinations.


Configuring hekad
-----------------------

A hekad configuration file specifies what inputs, decoders, filters, and outputs will be loaded. The configuration file is in `TOML <https://github.com/mojombo/toml>`_ format.
TOML looks is very similar to INI configuration formats, but with slightly more rich data structures and nesting support.

The config file is broken into sections, with each section representing a single instance of a plugin. The section name specifies the name of the plugin, and the “type” parameter specifies the plugin type;
this must match one of the types registered via the *pipeline.RegisterPlugin* function.

::

    [tcp:5565]
    type="TcpInput"
    parser_type="message.proto"
    decoder="ProtobufDecoder"
    address=":5565"


If you choose a plugin name that also happens to be a plugin type name, then you can omit the “type” parameter from the section and the specified name will be used as the type.
Thus, the following section describes a plugin named “TcpInput”, also of type “TcpInput”:

::

    [TcpInput]
    address = ":5566"
    parser_type = "message.proto"
    decoder = "ProtobufDecoder"


If a plugin fails to load during startup, hekad will exit at startup. When hekad is running, if a plugin should fail (due to connection loss, inability to write a file, etc.) then hekad will either shut down or restart the plugin if the plugin supports restarting.
When a plugin is restarting, hekad will likely stop accepting messages until the plugin resumes operation (this applies only to filters/output plugins).

Plugins specify that they support restarting by implementing the Restarting interface.

An internal diagnostic runner runs every 30 seconds to sweep the packs used for messages so that possible bugs in heka plugins can be reported and pinned down to a likely plugin(s) that failed to properly recycle the pack.


Global configuration options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can optionally declare a *[hekad]* section in your configuration file to configure some global options for the heka daemon.

Example hekad.toml file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    [hekad]
    cpuprof = "/var/log/hekad/cpuprofile.log"
    decoder_poolsize = 10
    max_message_loops = 4
    max_process_inject = 10
    max_timer_inject  = 10
    maxprocs = 10
    memprof = "/var/log/hekad/memprof.log"
    plugin_chansize = 10
    poolsize = 100

    # Listens for Heka messages on TCP port 5565.
    [TcpInput]
    address = ":5565"
    parser_type = "message.proto"
    decoder = "ProtobufDecoder"

    # Writes output from `CounterFilter`, `lua_sandbox`, and Heka's internal
    # reports to stdout.
    [debug]
    type = "LogOutput"
    message_matcher = "Type == 'heka.counter-output' || Type == 'heka.all-report' || Type == 'heka.sandbox-output'"

    # Counts throughput of messages sent from a Heka load testing tool.
    [CounterFilter]
    message_matcher = "Type == 'hekabench' && EnvVersion == '0.8'"
    output_timer = 1

    # Defines a sandboxed filter that will be written in Lua.
    [lua_sandbox]
    type = "SandboxFilter"
    message_matcher = "Type == 'hekabench' && EnvVersion == '0.8'"
    output_timer = 1
    script_type = "lua"
    preserve_data = true
    filename = "lua/sandbox.lua"
    memory_limit = 32767
    instruction_limit = 1000
    output_limit = 1024


Common Roles
^^^^^^^^^^^^^^

- **Agent** - Single default filter that passes all messages directly to another hekad daemon on a separate machine configured as an Router.
- **Aggregator** - Runs filters that can roll-up statistics (similar to statsd), and handles aggregating similar messages before saving them to a back-end directly or possibly forwarding them to a hekad router.
- **Router** - Collects input messages from multiple sources (including other hekad daemons acting as Agents), rolls up stats, and routes messages to appropriate back-ends.


Configuring Restarting Behavior
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Plugins that support being restarted have a set of options that govern how the restart is handled. If preferred, the plugin can be configured to not restart at which point hekad will exit,
or it could be restarted only 100 times, or restart attempts can proceed forever.

Adding the restarting configuration is done by adding a config section to the plugins’ config called *retries* . A small amount of jitter will be added to the delay between restart attempts.

Parameters:

- max_jitter (string)
- max_delay (string)
- delay (string)
- max_retries (int)

Inputs
^^^^^^^^^

**AMQPInput**

Connects to a remote AMQP broker (RabbitMQ) and retrieves messages from the specified queue. As AMQP is dynamically programmable, the broker topology needs to be specified in the plugin configuration.


**UdpInput**

Listens on a specific UDP address and port for messages. If the message is signed it is verified against the signer name and specified key version. If the signature is not valid the message is discarded
otherwise the signer name is added to the pipeline pack and can be use to accept messages using the message_signer configuration option.

Parameters:

- address(string): An IP address:port on which this plugin will listen.
- signer: Optional TOML subsection. Section name consists of a signer name, underscore, and numeric version of the key.
    - hmac_key (string): The hash key used to sign the message.
- decoder (string): A ProtobufDecoder instance must be specified for the message.proto parser. Use of a decoder is optional for token and regexp parsers; if no decoder is specified the raw input data is available in the Heka message payload.
- parser_type (string):
    - token - splits the stream on a byte delimiter.
    - regexp - splits the stream on a regexp delimiter.
    - message.proto - splits the stream on protobuf message boundaries.
- delimiter (string): Only used for token or regexp parsers.
    Character or regexp delimiter used by the parser (default “\n”). For the regexp delimiter a single capture group can be specified to preserve the delimiter (or part of the delimiter). The capture will be added to the start or end of the message depending on the delimiter_location configuration.
- delimiter_location (string): Only used for regexp parsers.
    - start - the regexp delimiter occurs at the start of the message.
    - end - the regexp delimiter occurs at the end of the message (default).

Example:

::

    [UdpInput]
    address = "127.0.0.1:4880"
    parser_type = "message.proto"
    decoder = "ProtobufDecoder"

    [UdpInput.signer.ops_0]
    hmac_key = "4865ey9urgkidls xtb0[7lf9rzcivthkm"
    [UdpInput.signer.ops_1]
    hmac_key = "xdd908lfcgikauexdi8elogusridaxoalf"

    [UdpInput.signer.dev_1]
    hmac_key = "haeoufyaiofeugdsnzaogpi.ua,dp.804u"


**TcpInput**

参数与UdpInput相似。

Example:

::

    [TcpInput]
    address = ":5565"
    parser_type = "message.proto"
    decoder = "ProtobufDecoder"

    [TcpInput.signer.ops_0]
    hmac_key = "4865ey9urgkidls xtb0[7lf9rzcivthkm"
    [TcpInput.signer.ops_1]
    hmac_key = "xdd908lfcgikauexdi8elogusridaxoalf"

    [TcpInput.signer.dev_1]
    hmac_key = "haeoufyaiofeugdsnzaogpi.ua,dp.804u"


**LogfileInput**

Tails a single log file, creating a message for each line in the file being monitored. Files are read in their entirety, and watched for changes. This input gracefully handles log rotation via the file moving but may lose a few log lines if using the “truncation” method of log rotation.
It’s recommended to use log rotation schemes that move the file to another location to avoid possible loss of log lines.

In the event the log file does not currently exist, it will be placed in an internal discover list, and checked for existence every discover_interval milliseconds (5000ms or 5s by default).

A single LogfileInput can only be used to read a single file. If you have multiple identical files spread across multiple directories (e.g. a */var/log/hosts/<HOSTNAME>/app.log* structure,
where each *<HOSTNAME>* folder contains a log file originating from a separate host), you’ll want to use the `LogfileDirectoryManagerInput <https://hekad.readthedocs.org/en/latest/man/plugin.html#config-logfile-directory-manager-input>`_.

Parameters:

- logfile (string): Each LogfileInput can have a single logfile to monitor.
- hostname (string): The hostname to use for the messages, by default this will be the machines qualified hostname. This can be set explicitly to ensure its the correct name in the event the machine has multiple interfaces/hostnames.
- discover_interval (int): During logfile rotation, or if the logfile is not originally present on the system, this interval is how often the existence of the logfile will be checked for. The default of 5 seconds is usually fine. This interval is in milliseconds.
- stat_interval (int): How often the file descriptors for each file should be checked to see if new log data has been written. Defaults to 500 milliseconds. This interval is in milliseconds.
- logger (string): Each LogfileInput may specify a logger name to use in the case an error occurs during processing of a particular line of logging text. By default, the logger name is set to the logfile name.
- use_seek_journal (bool): Specifies whether to use a seek journal to keep track of where we are in a file to be able to resume parsing from the same location upon restart. Defaults to true.
- seek_journal_name (string): Name to use for the seek journal file, if one is used. Only refers to the file name itself, not the full path; Heka will store all seek journals in a *seekjournal* folder relative to the Heka base directory.
    Defaults to a sanitized version of the *logger* value (which itself defaults to the filesystem path of the input file). This value is ignored if *use_seek_journal* is set to false.
- resume_from_start (bool): When heka restarts, if a logfile cannot safely resume reading from the last known position, this flag will determine whether hekad will force the seek position to be 0 or the end of file. By default, hekad will resume reading from the start of file.
- decoder (string)
- parser_type (string)
- delimiter (string)
- delimiter_location (string)


::

    [LogfileInput]
    logfile = "/var/log/opendirectoryd.log"
    logger = "opendirectoryd"


::

    [LogfileInput]
    logfile = "/var/log/opendirectoryd.log"


**LogfileDirectoryManagerInput**

Scans for log files in a globbed directory path and when a new file matching the specified path is discovered it will start an instance of the LogfileInput plugin to process it.
Each LogfileInput will inherit its configuration from the manager’s settings with the logfile property properly adjusted.

Parameters: (identical to LogfileInput with the following exceptions)

- logfile (string): A path with a globbed directory component pointing to a common (statically named) log file. Note that only directories can be globbed; the file itself must have the same name in each directory.
- seek_journal_name (string): With a LogfileInput it is possible to specify a particular name for the seek journal file that will be used. This is not possible with the LogfileDirectoryManagerInput;
    the seek_journal_name will always be auto- generated, and any attempt to specify a hard coded seek_journal_name will be treated as a configuration error.
- ticker_interval (uint): Time interval (in seconds) between directory scans for new log files. Defaults to 0 (only scans once on startup).

::

    [vhosts]
    type = "LogfileDirectoryManagerInput"
    logfile = "/var/log/vhost/*/apache.log"


**StatsdInput**

**StatAccumInput**

**ProcessInput**

Executes one or more external programs on an interval, creating messages from the output. If a chain of commands is used, stdout is piped into the next command’s stdin.
In the event the program returns a non-zero exit code, ProcessInput will stop, logging the exit error.

Each command is defined with the following parameters:

- name (string): Each ProcessInput must have a name defined for logging purposes. The messages will be tagged with name.stdout or name.stderr in the ProcessInputName field of the heka message.
- command (map[uint]cmd_config): he command is a structure that contains the full path to the binary, command line arguments, optional enviroment variables and an optional working directory.
    See the *cmd_config* definition below. ProcessInput expects the commands to be indexed by integers starting with 0.
- ticker_interval (uint): The number of seconds to wait between runnning *command* . Defaults to 15. A ticker_interval of 0 indicates that the command is run once.
- stdout (bool): Capture stdout from *command* . Defaults to true.
- stderr (bool): Capture stderr from *command* . Defaults to false.
- decoder (string): Name of the decoder instance to send messages to. Default is to inject messages back into the main heka router.
- parser_type (string):
    - token - splits the log on a byte delimiter (default).
    - regexp - splits the log on a regexp delimiter.
- delimiter (string): Only used for token or regexp parsers.
- delimiter_location (string): Only used for regexp parsers.
- timeout (uint): Timeout in seconds before any one of the commands in the chain is terminated.
- trim (bool) : Trim a single trailing newline character if one exists. Default is true.

cmd_config structure:

- bin (string): The full path to the binary that will be executed.
- args ([]string): Command line arguments to pass into the executable.
- environment ([]string): Used to set environment variables before *command* is run. Default is nil, which uses the heka process’s environment.
- directory (string): Used to set the working directory of *Bin* Default is “”, which uses the heka process’s working directory.


::

    [ProcessInput]
    name = "DemoProcessInput"
    ticker_interval = 2
    parser_type = "token"
    delimiter = " "
    stdout = true
    stderr = false
    trim = true

    [ProcessInput.command.0]
    bin = "/bin/cat"
    args = ["../testsupport/process_input_pipes_test.txt"]

    [ProcessInput.command.1]
    bin = "/usr/bin/grep"
    args = ["ignore"]


**HttpInput**

Starts a HTTP client which intermittently polls a URL for data. The entire response body is parsed by a decoder into a pipeline pack.
Data is always fetched using HTTP GET and any status code 200 generates a message with a configurable “sucess severity” (defaults to 6 or “information”).
Any non-200 status generate a message with a configurable “error severity” (defaults to 1 or “alert”) and are not fatal for the plugin.

Errors returned from HTTP GET such as inability to connect to remote host generate a message with a configurable “error severity” (defaults to 1 or “alert”) and of Type “heka.httpinput.error”.

Parameters:

- url (string): A HTTP URL which this plugin will regularly poll for data. This option cannot be used with the urls option. No default URL is specified.
- urls (array): An array of HTTP URLs which this plugin will regularly poll for data. This option cannot be used with the url option. No default URLs are specified.
- ticker_interval (uint): Time interval (in seconds) between attempts to poll for new data. Defaults to 10.
- success_severity (uint): Severity level of successful HTTP GET. Defaults to 6 (information).
- error_severity (uint): Severity level of errors, unreachable connections, and non-200 responses of successful HTTP GET. Defaults to 1 (alert).
- decoder (string): The name of the decoder used to transform the response body text into a structured hekad message. No default decoder is specified.


::

    [HttpInput]
    url = "http://localhost:9876/"
    ticker_interval = 5
    success_severity = 6
    error_severity = 1
    decoder = "ProtobufDecoder"


Decoders
^^^^^^^^^^^

**ProtobufDecoder**

**PayloadRegexDecoder**

Decoder plugin that accepts messages of a specified form and generates new outgoing messages from extracted data, effectively transforming one message format into another.

**PayloadJsonDecoder**

This decoder plugin accepts JSON blobs and allows you to map parts of the JSON into Field attributes of the pipeline pack message using JSONPath syntax.

**PayloadXmlDecoder**

This decoder plugin accepts XML blobs in the message payload and allows you to map parts of the XML into Field attributes of the pipeline pack message using XPath syntax using the `xmlpath <https://launchpad.net/xmlpath>`_ library.

**StatsToFieldsDecoder**

**MultiDecoder**

This decoder plugin allows you to specify an ordered list of delegate decoders. The MultiDecoder will pass the PipelinePack to be decoded to each of the delegate decoders in turn until decode succeeds.
In the case of failure to decode, MultiDecoder will return an error and recycle the message.

**Sandbox Decoder**

The sandbox decoder provides an isolated execution environment for data parsing and complex transformations without the need to recompile Heka.


Common Filter / Output Parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- message_matcher (string, optional): Boolean expression, when evaluated to true passes the message to the filter for processing. Defaults to matching nothing. See: `Message Matcher Syntax <https://hekad.readthedocs.org/en/latest/message_matcher.html#message-matcher>`_
- message_signer (string, optional): The name of the message signer. If specified only messages with this signer are passed to the filter for processing.
- ticker_interval (uint, optional): Frequency (in seconds) that a timer event will be sent to the filter. Defaults to not sending timer events.


Filters
^^^^^^^^^^^

**CounterFilter**

Once a second a CounterFilter will generate a message of type *heka.counter-output* . The payload will contain text indicating the number of messages that matched the filter’s *message_matcher* value during that second (i.e. it counts the messages the plugin received).
Every ten seconds an extra message (also of type *heka.counter-output* ) goes out, containing an aggregate count and average per second throughput of messages received.

Parameters: **None**

Example:

::

    [CounterFilter]
    message_matcher = "Type != 'heka.counter-output'"


**StatFilter**

**SandboxFilter**

**SandboxManagerFilter**


Outputs
^^^^^^^^^^^

**AMQPOutput**

**LogOutput**

Logs messages to stdout using Go’s *log* package.

**FileOutput**

Writes message data out to a file system.

Parameters:

- path (string): Full path to the output file.
- format (string, optional): Output format for the message to be written. Supports *json* or *protobufstream* , both of which will serialize the entire *Message* struct, or *text*, which will output just the payload string. Defaults to ``text``
- prefix_ts (bool, optional): Whether a timestamp should be prefixed to each message line in the file. Defaults to ``false`` .
- perm (string, optional): File permission for writing. A string of the octal digit representation. Defaults to “644”.

Example:

::

    [counter_file]
    type = "FileOutput"
    message_matcher = "Type == 'heka.counter-output'"
    path = "/var/log/heka/counter-output.log"
    prefix_ts = true
    perm = "666"


**TcpOutput**

Output plugin that serializes messages into the Heka protocol format and delivers them to a listening TCP connection. Can be used to deliver messages from a local running Heka agent to a remote Heka instance set up as an aggregator and/or router.

Parameters:

- address (string): An IP address:port to which we will send our output data.

::

    [aggregator_output]
    type = "TcpOutput"
    address = "heka-aggregator.mydomain.com:55"
    message_matcher = "Type != 'logfile' && Type != 'heka.counter-output' && Type != 'heka.all-report'"


**DashboardOutput**

Specialized output plugin that listens for certain Heka reporting message types and generates JSON data which is made available via HTTP for use in web based dashboards and health reports.


**ElasticSearchOutput**

Output plugin that serializes messages into JSON structures and uses HTTP requests to insert them into an ElasticSearch database.


**WhisperOutput**

**NagiosOutput**

**CarbonOutput**

**SmtpOutput**

Outputs a Heka message in an email. The message subject is the plugin name and the message content is controlled by the payload_only setting. The primary purpose is for email alert notifications i.e., PagerDuty.

Parameters:

- payload_only (bool): If set to true, then only the message payload string will be emailed, otherwise the entire Message struct will be emailed in JSON format. (default: true)
- send_from (string): email address of the sender
- send_to (array of strings): array of email addresses to send the message to
- host (string): SMTP host to send the email to
- auth (string): SMTP authentication type: “none”, “Plain”, “CRAMMD5” (default: “none”)
- user (string, optional): SMTP user name
- password (string, optional): SMTP user password