package l2f.gameserver.integrations;

import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

import l2f.gameserver.ThreadPoolManager;

/**
 * @author Invoke
 *
 */
public enum TwitchRewardManager
{
	INSTANCE;
	
	private static String apiRootUrl = "https://api.twitch.tv/";
	private static String krakenApiUrl = apiRootUrl + "kraken/";
	private static String streamsApiUrl = krakenApiUrl + "streams/";	
	private static String apiClientId = "";	
	private static int rewardTaskIntervalms = 60 * 100 * 120000;
	private static int reminderTaskIntervalms = 60 * 100 * 60000;
	private static int viewersForReward = 200;
	private static int rewardId = 57;
	private static int rewardCount = 10000;	

	public static void getInstance()
	{
		ThreadPoolManager.getInstance().scheduleAtFixedRate(new Runnable()
		{
			@Override
			public void run()
			{

			}
		}, rewardTaskIntervalms /2 , rewardTaskIntervalms);
	}
	
	public static void getInstance2()
	{
		ThreadPoolManager.getInstance().scheduleAtFixedRate(new Runnable()
				{
			@Override
			public void run()
			{
			}
				}, reminderTaskIntervalms /2 , reminderTaskIntervalms);
	}
	
	// tested but not working
	
	//public void initialise() 
	//{
	//	startReminderTask();
	//	startRewardTask();
	//}

	//private void startRewardTask()
	//{
		//ThreadPoolManager.getInstance().scheduleAtFixedRate(new TwitchRewardTask(streamerUsername), rewardTaskIntervalms, rewardTaskIntervalms);
	//}
	
	//private void startReminderTask()
	//{
	//	ThreadPoolManager.getInstance().scheduleAtFixedRate(new TwitchRewardReminderTask(streamerUsername), reminderTaskIntervalms, reminderTaskIntervalms);
	//}

	public boolean isStreaming(String apiResponse) 
	{
		if(apiResponse.length() == 0)
			return false;
		
		JsonObject streamObject = getStreamObjectFromApiResponse(apiResponse);
		
        return streamObject != null;
	}
	
	public int getViewerCount(String apiResponse) 
	{
		if(apiResponse.length() == 0)
			return 0;
		
		JsonObject streamObject = getStreamObjectFromApiResponse(apiResponse);
        
        if(streamObject == null)
        	return 0;
        
        int viewCount = streamObject.get("viewers").getAsInt();
        return viewCount;
	}
	
	public String getChannelName(String apiResponse) 
	{
		JsonObject channelObject = getChannelObject(apiResponse);
		return channelObject.get("display_name").getAsString();
	}
	
	public String getChannelUrl(String apiResponse) 
	{
		JsonObject channelObject = getChannelObject(apiResponse);
		return channelObject.get("url").getAsString();
	}
	
	private JsonObject getChannelObject(String apiResponse) 
	{
		if(apiResponse.length() == 0)
			return null;
		
		JsonObject streamObject = getStreamObjectFromApiResponse(apiResponse);
        
        if(streamObject == null)
        	return null;
        
        if(!streamObject.get("channel").isJsonObject())
        	return null;
        
        JsonObject channelObject = streamObject.get("channel").getAsJsonObject();
        
        return channelObject;
	}
	
	public String getStreamForUser(String username) 
	{
		try
		{
			return getApiResponseFromGET(getStreamUrlForUser(username));
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
		return "";
	}
	
	public int getViewersForReward() 
	{
		return viewersForReward;
	}
	
	public int getRewardId() 
	{
		return rewardId;
	}
	
	public int getRewardCount() 
	{
		return rewardCount;
	}
	
	private String getApiResponseFromGET(String url) throws IOException 
	{
		URL obj = new URL(url);
		HttpURLConnection con = (HttpURLConnection) obj.openConnection();
		con.setRequestMethod("GET");
		con.setRequestProperty("Client-ID", apiClientId);
		int responseCode = con.getResponseCode();
		if (responseCode == HttpURLConnection.HTTP_OK) { 
			BufferedReader in = new BufferedReader(new InputStreamReader(
					con.getInputStream()));
			String inputLine;
			StringBuffer response = new StringBuffer();

			while ((inputLine = in.readLine()) != null) {
				response.append(inputLine);
			}
			in.close();

			return response.toString();
		} 		
		return "";
	}
	
	private String getStreamUrlForUser(String username) 
	{
		return streamsApiUrl + username; 
	}	
	
	private JsonObject getStreamObjectFromApiResponse(String apiResponse)
	{
		JsonElement jelement = new JsonParser().parse(apiResponse);
        JsonObject  jobject = jelement.getAsJsonObject();
        
        if(jobject.get("stream").isJsonObject()) {
        	JsonObject streamObject = jobject.get("stream").getAsJsonObject();
    		return streamObject;
        }
        return null;
	}	
}
