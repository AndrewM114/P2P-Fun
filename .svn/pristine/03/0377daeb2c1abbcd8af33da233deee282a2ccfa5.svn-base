package Version1p1;
import java.io.BufferedReader;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.io.UnsupportedEncodingException;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketTimeoutException;
import java.util.ArrayList;
import java.util.Scanner;
import java.util.concurrent.ThreadLocalRandom;

//import Peer.State;

public class Version1p1 {
	Socket controlConnection = null;
	Socket peerClientConn = null;
	//input from server
	PrintWriter outToServer = null;
    BufferedReader inFromServer = null;
	
	DataInputStream inFromPeer = null;
	DataOutputStream outToPeer = null;
	//user input
	BufferedReader stdIn = null;
	
	Boolean hasServerSocket = false;
	ServerSocket peerConnection = null;
	
	
	String server_response;
	String user_input;
	String requested_file;
	
	int peer_port;
	int IP;
	
	boolean ready_sent = false;
	
	ArrayList<String> full_files = new ArrayList<String>();
	
	
	public enum State {
		STARTUP,
	    CONNECTED,
	    DOWNLOADING,
	    PEERSERVERSHARING,
	    PEERCLIENTSHARING,
	    SERVERSOCKETSETUP
	}
	
	
	
	
	/*
	 * In phase 1.1, we have a central server that manages 
	 * the peers sharing a specific file.  Each peer opens 
	 * a socket connection to the server (called a control connection) 
	 * for the purposes of exchanging string based commands. 
	 */
	
	
	//gameplan: (subject to change)
	//The protocol will be built upon the same basic protocol I used for the client/server project,
	//but will build in complexity from that
	//basically I will have methods that represent different states the peer is, 
	//and I will have methods that represent actual action that the peers take like 
	//transferFile, receiveFile, sendRequest, etc.
	//client needs to always be listening for new requests/connections, 
	//and always with a connection to the server
	
	
	//peer will work with at least 2 sockets. One for a peerToServer socket and 
	//one a peerToPeer socket
	//this way the client will be able to interact with the server through a socket while at the same
	//time interacting with a peer.
	//there is a simple state diagram in of the peers' behavior in the assignment doc
	
	
	//peer will need to be listening for a Request <filename> from the Server at all times
	//if it gets one, it will need to open a ServerSocket at a random
	//port (within the proper bounds, between 1500 and 2500 maybe?) and send that port number
	//back to the Server if it has the file, and a negative response if it doesn't have the file
	//it will then listen on the ServerSocket for another peer to connect, and when it does, this is
	//is when the file transfer happens.
	
	
	public void run() {
		
		BufferedReader stdIn = new BufferedReader(new InputStreamReader(System.in));
	    String userInput = "";
		
		State state = State.STARTUP;
		
		while (true) {		
			
			while (state== State.STARTUP) {
				state = startupState();
				//perhaps as time goes on I can change this to something like this:
				//state = startupState();
				//I will have to modify and clean up some code to make this work with the others
				
			}
			while (state == State.CONNECTED) {
				state = connectedState(userInput);
			}
			while (state == State.DOWNLOADING) {
				state = downloadingState();
			}
			//basically when SHARING and/or SERVERSOCKETSETUP is finished, it kicks back to CONNECTED where it continues to listen
			 //for userinput or server request
		
			while (state == State.PEERSERVERSHARING) {
				state = peerServerSharingState();
				
				
			}
			while (state == State.PEERCLIENTSHARING) {
				state = peerClientSharingState();
				
			}
			while (state == State.SERVERSOCKETSETUP) {
				System.out.println("inside SERVERSOCKETSETUP while loop");
				//serverSocketSetupState(state);
				state = serverSocketSetupState();
						
			}		
		}	
	}
	
	
	//code that runs during specific states
	
	public State startupState(){
		if (startUp()) { //this initializes variables that are global and will (maybe) give a list of files available
			//state = State.CONNECTED;
			System.out.println("State: CONNECTED");
			System.out.println("Please enter a request.");
			return State.CONNECTED;
			
		}
		else {
			System.out.println("Startup failed.");
			System.exit(0);
		}
		return State.STARTUP;
	}
	
	public State connectedState(String userInput) {
		//System.out.println("connected");
		//I think this needs to be here so it is reinitialized every time so that the condition if (stdIn.ready()) works
		//I can either reinit every time or I can init once before the loop, not sure which is better (or if either are valid)
		//if I can init once and still use StdIn.ready() or equiv, then that is better.
	
		//disregard the 3 comments above (for now) perhaps this will work this way??
	
		//during this state the peer needs to be simultaneously listening for both:
		//1. user_input
		//2. request from server
	
	
		//user_input = stdIn.readLine();
		//check for user input and act accordingly
		
		try {
			//System.out.println("is this working?");
			if(stdIn.ready()) {
				try {
					userInput = stdIn.readLine();
					Scanner ui = new Scanner (userInput); //TODO temp
					try {
						requested_file = ui.findInLine("[^Complete ].*"); // TODO
						initializeFile(requested_file);
					} catch (Exception e) {
						//System.out.println("requested_file was not saved");
						//do nothing
					}
					outToServer.println(userInput);
					outToServer.flush();
					
					stdIn = new BufferedReader(new InputStreamReader(System.in));
					
					//the server response is processed in the DOWNLOADING state
					//state = State.DOWNLOADING;
					return State.DOWNLOADING;
					
					//continue;
				} catch (IOException e) {
					e.printStackTrace();
				}
				//re-initialize this so that it doesn't get read twice when the loop iterates again
				stdIn = new BufferedReader(new InputStreamReader(System.in));
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		//check for server response and act accordingly
		try {
			if (inFromServer.ready()) {
				//process Server request somehow . . . 
				System.out.println("Received a request from server.");
				String from_server = inFromServer.readLine();
				if(from_server.contains("Request")) {
					//format should be Request <filename> eventually, but
					 //for testing purposes I will probably just assume one file for now
					
					String file_name = "";
					Scanner sc_request = new Scanner (from_server);
					file_name = sc_request.findInLine("[^Request ].*");
					
					//re-initializes this to prevent it from getting read twice on a second iteration 
					 //-- do this on proxy server assignment too (if this actually works the way I think it will)
					inFromServer = new BufferedReader(new InputStreamReader(controlConnection.getInputStream()));
					
					if (doesHave(file_name) == true) {
						requested_file = file_name;
						System.out.println("State:: SERVERSOCKETSETUP");
						//state = State.SERVERSOCKETSETUP;
						System.out.println("DO HAVE");
						return State.SERVERSOCKETSETUP;
					}
					else if (doesHave(file_name) == false) {
						outToServer.println("NO HAVE");
						outToServer.flush();
						System.out.println("NO HAVE");
						System.out.println("before connected state change");
						return State.CONNECTED;
					}
					
				}
				
			}
		} catch (IOException e) {
			
			e.printStackTrace();
		}
		return State.CONNECTED;
	}
	
	public State downloadingState() {
		try {
			server_response = inFromServer.readLine(); //not sure if readLine() is sufficient, or if I will need to scan and parse several lines
			System.out.println(server_response);
			
			//perhaps do a check to see if the server_response is in the correct format
			//if the server_response looks right set state to sharing
			if (server_response.equals("UNAVAILABLE") || server_response.equals("BAD REQUEST")) {
				//state = State.CONNECTED;
				System.out.println("Unable to perform request.");
				return State.CONNECTED;
			}
			if (server_response.equals("SUCCESSFUL")) {
				//state = State.CONNECTED;
				System.out.println("'Complete' Request Successful");
				return State.CONNECTED;
			}
			else {
				//The server sends back a response of the form ipaddress:port.
				//file_name = sc_request.findInLine("[^Version ].*");
				//Scanner scanforip = new Scanner(server_response);
				//Scanner scanforport = new Scanner(server_response);
				//String ip = scanforip.findInLine(".[^:]");
				//String port = sr.next();
				System.out.println("server_response " + server_response);
				String [] split = server_response.split("\\:");
				String ip = split[0];
				String port = split[1];
				
				String [] ipsplit = ip.split("/");
				
				System.out.println("ip: "+ipsplit[1]);
				System.out.println("port: "+port);
				System.out.println(System.currentTimeMillis()); 
				if (connectToPeer(ipsplit[1], port) == true) {
					System.out.println("State = State.PEERCLIENTSHARING");
					//state = State.PEERCLIENTSHARING;
					return State.PEERCLIENTSHARING;
				}
				else {
					System.out.println("Could not connect to peer./n State: CONNECTED");
					return State.CONNECTED;
					//state = State.CONNECTED;
					//break;
				}
				
			}
			
			//break;
			
			//if the server_response doesn't make sense, perhaps do a download failed event
				//System.out.println("Download failed or something");
				//state = State.CONNECTED;
		} catch (IOException e) {
			// TODO Auto-generated catch block
			System.out.println("Download failed");
			return State.CONNECTED;
		//e.printStackTrace();
		}
		finally {
			try {
				inFromServer = new BufferedReader(new InputStreamReader(controlConnection.getInputStream()));
			} catch (IOException e) {
				// TODO Auto-generated catch block
				//e.printStackTrace();
				System.out.println("inFromServer reinit failed");
			}
		}
	}
	
	public State peerClientSharingState() {
			saveFile(inFromPeer, requested_file);
			return State.PEERCLIENTSHARING;
	}
	
	public State peerServerSharingState() {
		try {
			System.out.println("Listening on port " +peerConnection.getLocalPort());
			
			System.out.println(System.currentTimeMillis()); 
			if (!ready_sent) {
				//ipaddress:port
				String message;
				InetAddress ip;
				String ips;
				try {
					//ip = InetAddress.getLocalHost();
					//ip = InetAddress.getLocalHost();
					ip = InetAddress.getLoopbackAddress();
					//ip = "127.0.0.1";
				} catch(Exception e) {
					//TODO make sure this is right
					ip = InetAddress.getLoopbackAddress();
				}
				message = ip.toString() + ":" + peer_port;
				outToServer.println(message);
				outToServer.flush();
				ready_sent = true;
				System.out.println(System.currentTimeMillis()); 
			}
			Socket peerClient = peerConnection.accept();
			
			System.out.println("peerConnection accepted");
			inFromPeer = new DataInputStream(peerClient.getInputStream());
			outToPeer = new DataOutputStream(peerClient.getOutputStream());
			
			
			//TODO PrintWriter works to send and it is received by the other peer through a DataInputStream,
			 //but as of now I can't get DataOutputStream to work
			
			sendFile(outToPeer, requested_file);
			
			//TODO as of now this isn't working because I don't have the code in place to set up the 
			 
		} 
		catch (SocketTimeoutException e) {
    		/* do nothing, this just means no client tried to connect */
		}
    		catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return State.PEERSERVERSHARING;
			//break;
		}
		return State.PEERSERVERSHARING; //TODO TEMP will need changed
	}
	
	public State serverSocketSetupState() {
		//get random port number
		System.out.println("serverSocketSetupState() called");
		peer_port = ThreadLocalRandom.current().nextInt(2500, 2700 + 1);
		try {
			peerConnection = new ServerSocket(peer_port);	
			//state = State.PEERSERVERSHARING;
			System.out.println("ServerSocketSetup Successful");
			return State.PEERSERVERSHARING;
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			System.out.println("ServerSocketSetup failed");
			//state = State.CONNECTED;
			return State.CONNECTED;
		}	
	}
	
	
	
	
	
	
	//////////////////////////////////who knows wut all this stuff below does /////////////////////////
	
	
	public boolean startUp() {
		try {
			controlConnection = new Socket("localhost", 1689);
			//inFromServer = new DataInputStream(controlConnection.getInputStream());
			//outToServer = new DataOutputStream(controlConnection.getOutputStream());
			
			outToServer = new PrintWriter(controlConnection.getOutputStream(), true);
		    inFromServer = new BufferedReader(new InputStreamReader(controlConnection.getInputStream()));
			
			stdIn = new BufferedReader(new InputStreamReader(System.in));
			//initializeFiles(); //this may be unnecessary in the long run, for now it simply initializes shareable files for testing purposes
			return true;
		} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
				return false;
		} 
		
		
	}
	//in the long term will be obsolete me thinks ... mainly used for testing purposes for sharing of files
	public void initializeFile(String filename) {
		String dir_path = "./Users/andrew/Documents/P2PProjFiles/Peer1 Files/"; //dot or no dot?
		
		File file = new File(dir_path + filename);
		
		PrintWriter writer;
		try {
			//perhaps a better "file management system" is in order than what I have below. This will do for now
			writer = new PrintWriter(file, "UTF-8");
			writer.println("Initial File");
			writer.println("This is the file to be shared, it will be in the initial folder");
			writer.println(" of the peer sharing it, and the receive folder of the peer that it's being shared with");
			writer.println(" (but of course only if the file transfer is successful).");
			writer.close();
		} catch (FileNotFoundException | UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		/* creating a byte file
		 * https://stackoverflow.com/questions/2885173/how-do-i-create-a-file-and-write-to-it-in-java
		byte data[] = ;
				FileOutputStream out = new FileOutputStream("the-file-name");
				out.write(data);
				out.close();
		*/
	}
	//maybe param should be a different type than String, String will have to do for now
	public boolean writeReceivedFile(String downloaded_file){
			
		File file = new File("/Users/andrew/Documents/P2PProjFiles/Peer2 Files/Received/file_to_share.txt");
		
		PrintWriter writer;
		try {
			writer = new PrintWriter(file, "UTF-8");
			writer.print(downloaded_file);
			writer.close();
			
			return true;
		} catch (FileNotFoundException | UnsupportedEncodingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return false;
		}
		
		
	}
	public void processServerResponse(DataInputStream inFromServer){
		
	}
	//checks if StdIn contains userInput, if so processes it and sends it to the server
	public void processStdIn(BufferedReader StdIn) {
		try {
			if (stdIn.ready()) {
				String userInput;
				
				//processUserInput 
				userInput = stdIn.readLine();
				sendUserInput(userInput);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public void sendUserInput(String input) {
		
	}
	public boolean doesHave(String filename) {
		//arraylist full_files is a list of all the complete files this peer has
		
		if (full_files.contains(filename)) {
			return true;
		}
		return false;
	}
	public boolean connectToPeer(String ip, String port) {

		
		try {
			peerClientConn = new Socket(ip, Integer.parseInt(port));
			inFromPeer = new DataInputStream(peerClientConn.getInputStream());
			outToPeer = new DataOutputStream(peerClientConn.getOutputStream());
			System.out.println("Connection to peer successful");
			return true;
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			//e.printStackTrace();
			System.out.println("Connection to peer failed");
			return false;
		}
		
	}

	public void complete_report(PrintWriter outToServer) {
		
	}
	
	//TODO get these to work
	//TODO
	//TODO
	//basically I can get stuff to transfer through PrintWriter and Buffered Reader but having trouble with DataOuput/InputStream
	//https://stackoverflow.com/questions/9520911/java-sending-and-receiving-file-byte-over-sockets
	//this link is helpful
	//Basically I need to work out some of the details of how to transfer these successfully, then when it's done
	 //to kick back to CONNECTED state without throwing any exceptions. Transfer is kind of working, but it's also transfering
	 //a hardcoded file instead of the file that is requested. Code needs to be written to handle this.
	 //Perhaps have something that has the server give you a list of files and then you can request one from a list of options
	 //then the check is made. This way you don't have to type in an entire filename. Also perhaps find a way to hard-code the 
	 //path to the peer's folder, but have a different folder for each peer, so it doesn't have to check the entire path, just the
	 //file name.
	public void sendFile(DataOutputStream outToPeer, String filename) throws FileNotFoundException {
		String msg = "Is this working?";
		//byte[] sendData = new byte[1024];
		//sendData = msg.getBytes();
		//outToPeer.write(sendData);
		//outToPeer.flush();
		String dir_path = "./Users/andrew/Documents/P2PProjFiles/Peer1 Files/"; //dot or no dot?
		String fn = "test_transfer.txt"; //TODO temp
		
		File file = new File(dir_path + filename);
		long length = file.length();
        byte[] bytes = new byte[16 * 1024];
        
        InputStream in = new FileInputStream(file);
			
        int count;
        try {
        		while ((count = in.read(bytes)) > 0) {
        			outToPeer.write(bytes, 0, count);
        		}
        } catch (Exception e) {
        	
        }
        
        try {
			outToPeer.close();
			in.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
	
	public void saveFile(DataInputStream inFromPeer, String filename) {
		
		FileOutputStream outToFile = null;
		
		String dir_path = "./Users/andrew/Documents/P2PProjFiles/Peer2 Files/"; //dot or no dot?
		String fn = "test_transfer.txt"; //TODO temp
		
		File file = new File(dir_path + filename);
		
		
		try {
            outToFile = new FileOutputStream(file);
        } catch (FileNotFoundException ex) {
            System.out.println("File not found. ");
        }
		int count;
		byte[] buffer = new byte[8192]; // or 4096, or more
		try {
			while ((count = inFromPeer.read(buffer)) > 0)
			{
				outToFile.write(buffer, 0, count);
			} 
		} catch(Exception e) {
				
			}
		
		
	}
	
}
	
	

	
	
