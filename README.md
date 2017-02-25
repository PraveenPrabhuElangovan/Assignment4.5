1)CheckSum and importance of checksum:
	A checksum or hash sum is a small-size datum from a block of digital data for the purpose of detecting errors which may have been introduced during its transmission or storage. • HDFS checksums all data written to it and by default verifies checksums when reading data. • A separate checksum is created for every dfs.bytes-per-checksum bytes of data. • The default is 512 bytes, and because a CRC-32C checksum is 4 bytes long. The storage overhead is less than 1%.Datanodes are responsible for verifying the data they receive before storing the data and its checksum. • When the clients read data from datanodes, they verify checksums as well, comparing them with the ones stored at the datanodes. • -get command does the checksum verification during the data read. • -copyFromLocal doesn’t perform checksum during data read. • -ignoreCrc option with the -get is equivalent to -copyToLocal command. • Disabling checksum verification is useful if we have a corrupt file that we want to inspect so that we can decide what to do with it

2)Anatomy of file write to HDFS:
a) Client creates, calls create() on DistributedFileSystem. 
b) DistributedFileSystem contacts namenode to create a new file in the filesystem’s namespace, with no blocks associated with it. The namenode performs various checks to make sure the file doesn’t already exist and that the client has the right permissions to create the file. If these checks pass, the namenode makes a record of the new file (in edits) 
c) DistributedFileSystem returns an FSDataOutputStream for the client to start writing data. FSDataOutputStream uses DFSOutputStream, which handles communication with the datanodes and namenode. 
d) Client signals write() method on FSDataOutputStream.   
e) DFSOutputStream splits data into packets and writes it to an internal queue called the data queue.
f) When the client has finished writing data, it calls close() on the stream. This flushes all the remaining packets to the datanode pipeline and waits for acknowledgments. 
g) DistributedFileSystem contacts the namenode to signal that the file write activity is complete.

3)HDFS handling failure while file writing:
a) The pipeline is closed and any packets in the ack queue are added to the front of the data queue. 
b) The current block on the good datanodes is given a new identity, which is communicated to the namenode 
c) The failed datanode is removed from the pipeline, and a new pipeline is constructed from the two good datanodes. 
d) The remainder of the block’s data is written to the good datanodes in the pipeline. 
e) The namenode notices that the block is under-replicated, and it arranges for a further replica to be created on another node. 
f) As long as dfs.namenode.replication.min replicas (which defaults to 1) are written, the write will succeed. 
g) The block will be asynchronously replicated across the cluster until its target replication factor is reached
