package Version1p1;



import java.io.*;
import java.net.*;
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


public class ServerV1p1 {
	
	//Vector clients = new Vector ();
	ArrayList<Socket> clientSockets = new ArrayList<Socket>();
	
	HashMap<String, FileInfo> file_record = new HashMap<String, FileInfo>();
	
	
	public void run() throws IOException {
		
		//server needs to keep track of all files kept by the peers
		//whenever a peer connects, perhaps give info about what type of files that client has and 
		//store in some sort of hashmap
		//when that file is requested, query the hashmap (or other data structure like a storage class? whatever it's called?)
		//and find out where the file is, who has it and return those details to the client who's asking
		
		ServerSocket serverSocket = null; 
        
        /*
         * Create the server socket and set a timeout so that
         * it will not wait forever for a client to connect when
         * we call accept ().
         */
        
        try { 
            	serverSocket = new ServerSocket(1689);
            	serverSocket.setSoTimeout (50);
            	System.out.println ("Listening on port 1689");
        } 
        catch (IOException e) { 
        		System.err.println("Could not listen on port: 1689."); 
        		System.exit(-1); 
        } 

        do {
        		/* 
        		 * Give new clients a chance to connect.
        		 */
        	
        		Socket client = null;
        	
        		try {
        			client = serverSocket.accept ();
        			System.out.println("peer accepted");
        		}
        		catch (SocketTimeoutException e) {
        		/* do nothing, this just means no client tried to connect */
			}
        		catch (Exception e) {
        			e.printStackTrace ();
        			break;
			}

        		/*
        		 * If we got a new client, add them to the vector/arraylist
        		 */
        		
        		if (client != null) {
        			clientSockets.add (client);
        			System.out.println("peer added to Arraylist");
        		}
        			
        	
        		/*
        		 * Then give all older clients a chance to do something.
        		 * Different strategies exist for the most efficient ways
        		 * of doing this (for example, checking for new clients 
        		 * before processing each old client, rather than before
        		 * the entire vector). 
        		 */
        	
        		//this little loop will need modified probably
        		
        		//I think what this loop does is go through the arraylist of peers and process any input from them
        		int i = 0;
        		while (i < clientSockets.size ())	{
        			//System.out.println("Processing peer "+ i);
        			if (processClientInput (clientSockets.get (i))) 
        				i++;
        		}
        	
        } while (true);
	
        try {
        		serverSocket.close();
		}
        catch (IOException e) {
        		e.printStackTrace ();
		}
    	
	}

	public boolean processClientInput(Socket client) {
		
		//I will probably need to change these to DataInputStream and DataOutputStream instead
		 //of BufferedReader and PrintWriter
		
		BufferedReader inFromClient = null;
		PrintWriter outToClient = null;
		
    		String input = null;
    	
	    try {
	    		inFromClient = new BufferedReader (new InputStreamReader (client.getInputStream ()));
	    		outToClient = new PrintWriter (client.getOutputStream (), true);
	    	
	    }
	    catch (Exception e) {
	    		e.printStackTrace ();
	    		clientSockets.remove(client);
	    		return false;
	    }
	
	    try {
	    		//with BufferedInputReader there was an if (inFromClient.ready()) condition here, 
	    		//but that doesn't exist with DataInputStream. Hopefully I can do without it
	    		//perhaps the if(scan_in.hasNext()) is a sufficient replacement for that
	    		if (inFromClient.ready()) {
	    			String request = "";
	    			String response = "";
	   		
	    			request = inFromClient.readLine(); //this is not firing off...
	    			System.out.println(request);
	    		
	    			response = processRequest(request, client);
	    			outToClient.println(response);
	    			outToClient.flush();
	    		}
	    		return true;
	   		
	    }  
	    catch (Exception e) {
	    		/*
	    		 * If we get an exception when communicating with a client,
	    		 * we should drop the client from the vector of clients, so 
	    		 * we don't waste time processing the same client later.
	    		 */
	    		clientSockets.remove(client);
	        e.printStackTrace ();
	        return false;
	    }
	    
	    //return true;
	}
	
	
	public String processRequest(String request, Socket peer) {
		System.out.println("Request: "+request);
		//I will need a way to store a record that can be referenced of the peers and their files
		
		String file_name = "";
		Scanner sc_request = new Scanner (request);
		
		
		//format: WhereIs <filename>
		
		if (request.matches("^(WhereIs).*")) {
			System.out.println("WhereIs requested");
			//find out where it is -- I think WhereIs is the beginning of the request, and the filename follows
			
			//The server here will send a request to the peer it thinks (knows) has the file
			
			//A hash map where
			 //the key is the filename, and the info is a class object that contains everything about the file, including
			 //a list of the peers that have the file
			
			file_name = sc_request.findInLine("[^WhereIs].*").replaceAll("\\s", ""); //this won't work if file name is nested between two literal arrows like this <filename>
			System.out.println("file_name: " + file_name);
			sc_request.close();
			
			for (String key : file_record.keySet()) {
			    System.out.println("Files:" + key);
			}
			if (file_record.containsKey(file_name)) {
				System.out.println("It contains the file as a key name");
				FileInfo fr = file_record.get(file_name);
				
				if (fr.owners.isEmpty()) {
					System.out.println("Owners arraylist for that file is empty");
					return "UNAVAILABLE";
				}
				else {
					//get random peer from owners arraylist
					//goes through every owner to see if they actually have it
					for (int i = 0; i < fr.owners.size(); i++) {
						
						Socket who_has_it = fr.owners.get(i);
						
						InetAddress ipaddress = who_has_it.getInetAddress();
						int port = who_has_it.getPort();
						//String response = ipaddress.toString() + ":" + port;
						//System.out.println(response);
					
						String moment_of_truth = requestConfirm(who_has_it, file_name);
						
						if (moment_of_truth.equals("UNAVAILABLE")) {
							return "UNAVAILABLE";
							//return moment_of_truth;
						}
						else if (moment_of_truth.equals("NO HAVE")) {
							fr.owners.remove(i);
							System.out.println("This socket didn't actually have the file, remove from records");
							i --; //redo this index, because the indices shifted // I think this will work...
						}
						else {
							try {
								Thread.sleep(1000);
							} catch (InterruptedException e) {
								// TODO Auto-generated catch block
								//e.printStackTrace();
								System.out.println("Sleep failed");
							}
							return moment_of_truth;
						}
					}
					return "UNAVAILABLE";
				}
			}
			else {
				return "UNAVAILABLE";
			}
		}
		//format: Complete <filename>  	--> perhaps not have literal arrows around the filename
		if (request.matches("^(Complete).*")) {	//prob has enough details to work, expand later
			System.out.println("Complete request called");
			
			//this is when the peer tells the server that it has a complete version of a file
			//this request does not require a response from the server
			//when the server gets this, it should save it in some sort of table or "database" that
			//it can reference later when asked WhereIs
			
			file_name = sc_request.findInLine("[^Complete].*").replaceAll("\\s", ""); //this won't work if file name is nested between two literal arrows like this <filename>
			
			System.out.println(file_name);
			sc_request.close();
			if (file_record.containsKey(file_name)) {
				FileInfo fr = file_record.get(file_name);
				fr.owners.add(peer);
				System.out.println("peer added to existing record for file");
			}
			else {
				FileInfo fr = new FileInfo();
				fr.file_name = file_name;
				fr.file_size = -1; //TODO: need to init this somehow. Perhaps the "Complete" request should give a file_size and version
				fr.version = "1.1";
				fr.md5 = "https://stackoverflow.com/questions/304268/getting-a-files-md5-checksum-in-java"; //https://stackoverflow.com/questions/304268/getting-a-files-md5-checksum-in-java
				fr.owners.add(peer);
				file_record.put(file_name, fr);
				System.out.println("new record created for file with peer as owner");
			}
			return "SUCCESSFUL";
			//prob has enough details to work, will need expansion and debugging
			
		}
		//format: FileInfo <filename> ? perhaps not with literal arrows '<' '>'
		if (request.matches("^(FileInfo).*")) {
			//not totally sure about this one. It's a little vague on all the details.
			//This is sent by the peer to fetch the information on the file being shared.  
			System.out.println("FileInfo request called");

			
			file_name = sc_request.findInLine("[^FileInfo ].*"); //this won't work if file name is nested between two literal arrows like this <filename>
			System.out.println(file_name);
			sc_request.close();
			
			if (file_record.containsKey(file_name)) {
				FileInfo fr = file_record.get(file_name);
				String response =  fr.md5 + ":" + fr.file_size + ":" + fr.file_name;
				System.out.println(response);
				return response;
			}
			else {
				return "UNAVAILABLE";
			}
			
			
			//The response is of the form md5:filesize:filename.  The md5 checksum and file size 
			//can be used to validate a completed download 
			//before sending the Complete message to the server.  
			
		}
		if (request.matches("^(Version).*")) {
			System.out.println("Version request called");

			//Unclear on exactly why this is required tbh.
			
			file_name = sc_request.findInLine("[^Version].*").replaceAll("\\s", ""); //this won't work if file name is nested between two literal arrows like this <filename>
			System.out.println(file_name);
			sc_request.close();
			
			if (file_record.containsKey(file_name)) {
				FileInfo fr = file_record.get(file_name);
				String response = fr.version;
				System.out.println(response);
				return response;
			}
			else {
				return "UNAVAILABLE";
			}
			
			
			
			//Version – This is sent by the peer to ask which version of the protocol 
			//the server understands.
			//The response is an arbitrary string that specifies the version.  
			//This version must exactly match what the peer understands 
			//in order for the peer to continue.


		}
		else {
			return "BAD REQUEST";  
		}
		
		
	}
	
	public String requestConfirm(Socket peer, String file_name) {
		//send a request to this peer
		//if peer is currently connected
		if (clientSockets.contains(peer)) {
			//if peer has the file
			BufferedReader inFromClient = null;
			PrintWriter outToClient = null;
			
    			try {
    				inFromClient = new BufferedReader (new InputStreamReader (peer.getInputStream ()));
				outToClient = new PrintWriter (peer.getOutputStream (), true);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
    			outToClient.println("Request "+ file_name);
    			outToClient.flush();
			String resp = "";
			
    			try {
					resp = inFromClient.readLine();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
    			return resp;
		}
		else {
			return "UNAVAILABLE";
		}
	}
	
}
