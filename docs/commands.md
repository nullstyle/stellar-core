stellar-core can be controlled via the following commands.

## Command line options
* **--?** or **--help**: Print the available command line options and then exit..
* **--c** Send an [HTTP command](#HTTP-Commands) to an already running local instance of stellar-core and then exit. For example: `> stellar-core -c info`
* **--conf FILE**: Specify a config file to use. You can use '-' and provide the config file via STDIN. *default 'stellar-core.cfg'*
* **--convertid ID**: Will output the passed ID in all known forms and then exit. Useful for determining the public key that corresponds to a given private key. For example:
`stellar-core --convertid SDQVDISRYN2JXBS7ICL7QJAEKB3HWBJFP2QECXG7GZICAHBK4UNJCWK2`
* **--dumpxdr FILE**:  Dumps the given XDR file and then exits.
* **--forcescp**: This command is used to start a network from scratch or when a network has lost quorum because of failed nodes or otherwise. It sets a flag in the database. The next time stellar-core is run, stellar-core will start emitting SCP messages based on its last known ledger rather than waiting to hear a ledger close from the network. This doesn't change the requirements for quorum so although this node will emit SCP messages SCP won't complete until there are also a quorum of other nodes also emitting SCP messages on this same ledger.
* **--fuzz FILE**: Run a single fuzz input and exit.
* **--genfuzz FILE**:  Generate a random fuzzer input file.
* **--genseed**: Generate and print a random public/private key and then exit.
* **--info**: Shortcut for `--c info`
* **--ll LEVEL**: Set the log level. It is redundant with `--c ll` but we need this form if you want to change the log level during test runs.
* **--metric METRIC**: Report metric METRIC on exit. Used for gathering a metric cumulatively during a test run.
* **--newdb**: Clears the local database and resets it to the genesis ledger. If you connect to the network after that it will catch up from scratch. 
* **--newhist ARCH**:  Initialize the named history archive ARCH. ARCH should be one of the history archives you have specified in the stellar-core.cfg. This will write a `.well-known/stellar-history.json` file in the archive root.
* **--test**: Run all the unit tests. For [further info](https://github.com/philsquared/Catch/blob/master/docs/command-line.md) on possible options for test. For example this will run just the "Herder" tests and stop after the first failure: `stellar-core --test -a [Herder]` 
* **--version**: Print version info and then exit.


## HTTP Commands
By default stellar-core listens for connections from localhost on port 11626. 
You can send commands to stellar-core via a web browser, curl, or using the --c 
command line option (see above). Most commands return their results in JSON format.

* **help**
  Prints a list of currently supported commands.

* **catchup** 
  `/catchup?ledger=NNN[&mode=MODE]`
  Triggers the instance to catch up to ledger NNN from history;
  Mode is either 'minimal' (the default, if omitted) or 'complete'.

* **checkpoint**
  Triggers the instance to write an immediate history checkpoint.

* **checkdb**
  Triggers the instance to perform a background check of the database's state.

* **checkpoint**
  Forces uploading a history checkpoint to the archive.

* **connect**
  `/connect?peer=NAME&port=NNN`
  Triggers the instance to connect to peer NAME at port NNN.

* **dropcursor**
  `/dropcursor?id=XYZ` deletes the tracking cursor with identified by `id`. See `setcursor` for more information.

* **info**
  Returns information about the server in JSON format (sync
  state, connected peers, etc).

* **ll**
 `/ll?level=L[&partition=P]`
  Adjust the log level for partition P (or all if no partition is specified).
  Level is one of FATAL, ERROR, WARNING, INFO, DEBUG, VERBOSE, TRACE

* **maintenance**
 `/maintenance?[queue=true]`
  Performs maintenance tasks on the instance.
   * `queue` performs deletion of queue data. See `setcursor` for more information.

* **metrics**
 Returns a snapshot of the metrics registry (for monitoring and
debugging purpose).

* **peers**
  Returns the list of known peers in JSON format.

* **setcursor**
 `/setcursor?id=ID&cursor=N`
  sets or creates a cursor identified by `ID` with value `N`. ID is an uppercase AlphaNum, N is an uint32 that represents the last ledger sequence number that the instance ID processed.
  Cursors are used by dependent services to tell stellar-core which data can be safely deleted by the instance.
  The data is historical data stored in the SQL tables such as txhistory or ledgerheaders. When all consumers processed the data for ledger sequence N the data can be safely removed by the instance.
  The actual deletion is performed by invoking the `maintenance` endpoint.

* **scp**
  Returns a JSON object with the internal state of the SCP engine.

* **tx**
  `/tx?blob=Base64`
  submit a [transaction] (/docs/concepts/transaction.md) to the network.
  blob is a base64 encoded XDR serialized 'TransactionEnvelope'
  returns a JSON object with the following properties
  status:
    * "PENDING" - transaction is being considered by consensus
    * "DUPLICATE" - transaction is already PENDING
    * "ERROR" - transaction rejected by transaction engine
        error: set when status is "ERROR".
            Base64 encoded, XDR serialized 'TransactionResult'

### The following HTTP commands are exposed on test instances
* **generateload**
  `/generateload[?accounts=N&txs=M&txrate=(R|auto)]`
  Artificially generate load for testing; must be used with `ARTIFICIALLY_GENERATE_LOAD_FOR_TESTING` set to true.

* **manualclose**
  If MANUAL_CLOSE is set to true in the .cfg file. This will cause the current ledger to close.

* **testacc**
 `/testacc?name=N`
 Returns basic information about the account identified by name. Note that N is a string used as seed, but "root" can be used as well to specify the root account used for the test instance.

* **testtx**
 `/testtx?from=F&to=T&amount=N&[create=true]
  Injects a payment transaction (or a create transaction if "create" is specified) from the account F to the account T, sending N XLM to the account.
  Note that F and T are seed strings but can also be specified as "root" as a shorthand for the root account for the test instance.
