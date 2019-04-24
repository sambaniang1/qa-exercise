// depending on the IDE/environment you use, your project files can reside here.

/// Samba Work 

package task;

import static io.restassured.RestAssured.given;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Map;
import java.util.TreeMap;

import org.testng.Assert;
import org.testng.annotations.Test;

import io.restassured.response.Response;

public class NSSTodoAutomation {

	// @Test
	public void checkStatusCode() {
		given().get("http://localhost:80/nss-todo-automation-alpha/fake-API-call.php").then().statusCode(200);
	}

	 //@Test
	public void printTasksWithNoCategory() {
		Response res = given().get("http://localhost:80/nss-todo-automation-alpha/fake-API-call.php");
		// .then().contentType(ContentType.JSON).extract().path("[*].category");
		String responseStr = res.asString();

		String searchedText = "category";

		ArrayList<String> valueList = getValueList(responseStr, searchedText);
		int categoryCount = 0;
		for (String value : valueList) {
			if (null != value && !"".equals(value)) {
				categoryCount++;
			}
		}

		System.out.println("Number of task without category empty is " + categoryCount);

	}

	//@Test
	public void printAllTaskNames() {
		Response res = given().get("http://localhost:80/nss-todo-automation-alpha/fake-API-call.php");
		String responseStr = res.asString();

		String searchedText = "task name";

		ArrayList<String> valueList = getValueList(responseStr, searchedText);
		
		for (String value : valueList) {
			if (null != value && !"".equals(value)) {
				System.out.println(value);
			}
		}
		
	}

	//We have assumed that the task without due date are printed first
	//@Test
	public void printTasksOrderByDueDate(){
		Response res = given().get("http://localhost:80/nss-todo-automation-alpha/fake-API-call.php");
		String responseStr = res.asString();
		
		//We have assumed that the task without due date are printed first
		ArrayList<String> dueDateList = getValueList(responseStr, "due date");
		ArrayList<String> taskNameList = getValueList(responseStr, "task name");
		Map<Long, String> taskDueMap = new TreeMap<Long, String>(Collections.reverseOrder());
		
		for(int i = 0;i<taskNameList.size();i++){
			String task = taskNameList.get(i);
			String dueDate = dueDateList.get(i).replace("\\r\\n", "");
			if(null != dueDate && !"".equals(dueDate)){
				taskDueMap.put(Long.parseLong(dueDate), task);
			}else{
				System.out.println(task);
			}
		}
		
		for(Long due: taskDueMap.keySet()){
			System.out.println(taskDueMap.get(due));
		}
	}
	
	@Test
	public void countAndValidateTasks(){
		Response res = given().get("http://localhost:80/nss-todo-automation-alpha/fake-API-call.php");
		String responseStr = res.asString();
		ArrayList<String> dueDateList = getValueList(responseStr, "due date");
		ArrayList<String> taskNameList = getValueList(responseStr, "task name");
		
		Assert.assertEquals(dueDateList.size(), taskNameList.size(), "Task names are not equal to the ids");
		System.out.println("No of tasks found is "+taskNameList.size()+" and it is same as no of ids");
		
	}
	
	
	public ArrayList<String> getValueList(String responseStr, String searchedText) {
		int searchedTextIndex = responseStr.indexOf(searchedText);

		ArrayList<String> valueList = new ArrayList<String>();

		while (-1 != searchedTextIndex) {
			int colonIndex = responseStr.indexOf(":", searchedTextIndex);
			int nextLineIndex = responseStr.indexOf('\n', colonIndex);
			String reqData = responseStr.substring(colonIndex + 1, nextLineIndex);
			String finalReq = reqData.replace("\"", "").replace(" ", "").replace(",", "");
			valueList.add(finalReq);
			searchedTextIndex = responseStr.indexOf(searchedText, nextLineIndex);
		}
		return valueList;
	}
}
