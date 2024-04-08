Pricing BRE Java
  
  package com.ugro.bre.pricing.api;

import com.zebrunner.carina.api.AbstractApiMethodV2;
import com.zebrunner.carina.api.annotation.Endpoint;
import com.zebrunner.carina.api.annotation.SuccessfulHttpStatus;
import com.zebrunner.carina.api.http.HttpMethodType;
import com.zebrunner.carina.api.http.HttpResponseStatusType;
import com.zebrunner.carina.utils.config.Configuration;

@Endpoint(url = "${base_url}/api/v1/run", methodType = HttpMethodType.POST)
@SuccessfulHttpStatus(status = HttpResponseStatusType.OK_200)
public class Pricing_Bre extends AbstractApiMethodV2 {

    public Pricing_Bre() {
        super("api/ugro_capital/pricing_bre_req.json", "api/ugro_capital/pricing_bre_res.json","properties/ugro_capital/pricing_bre_req.properties");
        replaceUrlPlaceholder("base_url", Configuration.getRequired("api_url"));
    }
}

  config.properties

  

#=============== Environment configuration ===========#
env=UAT
UAT.api_url=https://rules-uat.ugrocapital.com
UAT.api_endpoint=https://rules-uat.ugrocapital.com

UAT.Authorization =Bearer tD6tL7BpRIW9vNmvDBgZ

------------------------------------------------------------------------------------------------------------------------------
package com.ugro.bre.pricing.test;

import java.util.HashMap;

import org.testng.annotations.Test;
import org.testng.asserts.SoftAssert;
import com.ugro.bre.pricing.api.Pricing_Bre;
import com.ugro.utils.ExtentManager;
import com.ugro.utils.WriteResponse;
import com.ugro.utils.resonse;
import com.zebrunner.carina.core.IAbstractTest;
import com.zebrunner.carina.dataprovider.IAbstractDataProvider;
import com.zebrunner.carina.dataprovider.annotations.XlsDataSourceParameters;
import com.zebrunner.carina.utils.R;

import io.restassured.response.Response;

public class Ugro_Capital_Test extends resonse implements IAbstractTest, IAbstractDataProvider {

	static int rowNum = 1;
	WriteResponse writeReponse = new WriteResponse();
	String xlpath;
	String sheetName;
	
	@Test(dataProvider = "DataProvider")
	@XlsDataSourceParameters(path = "data_source/shakti_sl.xlsx", sheet = "Sheet1")
	public void bankingPricingBre(HashMap<String, String> testData) throws Exception {
        String status = "PASS";
        String failureReason = "";
		try {
			 xlpath = "D:\\Ugro_BRE_Automation\\src\\test\\resources\\data_source\\shakti_sl.xlsx";
			 sheetName = "Sheet1";
	
			Pricing_Bre pricingbrebank = new Pricing_Bre();
			
			//SoftAssert softAssert = new SoftAssert();
			pricingbrebank.getProperties().replace("ROI", testData.get("ROI"));
			pricingbrebank.getProperties().replace("LOAN_AMOUNT", testData.get("LOAN_AMOUNT"));
			pricingbrebank.getProperties().replace("Processing_Fees", testData.get("Processing_Fees"));
			pricingbrebank.getProperties().replace("Insurance_Penetrat", testData.get("Insurance_Penetrat"));
			pricingbrebank.getProperties().replace("CERSAI_Master", testData.get("CERSAI_Master"));
			pricingbrebank.getProperties().replace("Stamp_Dutity", testData.get("Stamp_Dutity"));
			pricingbrebank.getProperties().replace("Pre_Part", testData.get("Pre_Part"));
			pricingbrebank.getProperties().replace("Foreclosure", testData.get("Foreclosure"));
			pricingbrebank.getProperties().replace("NO_OF_PROPERTIES", testData.get("NO_OF_PROPERTIES"));
			pricingbrebank.getProperties().replace("Legal_Technical_Charges", testData.get("Legal_Technical_Charges"));
			pricingbrebank.getProperties().replace("Documentation_Charges", testData.get("Documentation_Charges"));
			pricingbrebank.setHeader("Authorization", R.CONFIG.get(R.CONFIG.get("env") + ".Authorization"));
	
			pricingbrebank.callAPI();
			Response res = pricingbrebank.callAPI();
			String resBody = res.getBody().asString();
			
			ExtentManager.logJson(resBody);
			writeReponse.writeResponse(xlpath, sheetName, 14, rowNum, resBody);
			pricingbrebank.setProperties("properties/ugro_capital/pricing_bre_req.properties");
			pricingbrebank.getProperties().replace("ExpectedResponse", testData.get("ExpectedResponse"));
			pricingbrebank.validateResponse();
			writeReponse.writeResponse(xlpath, sheetName, 15, rowNum, "PASS");
			ExtentManager.logPassDetails("Test case status : PASS");
		}
		catch(Exception e) {
			e.printStackTrace();
			failureReason = e.getMessage();
			
			status = "FAIL";
			ExtentManager.logFailDetails("Test case status :"+status);
			writeReponse.writeResponse(xlpath, sheetName, 15, rowNum, status);
			writeReponse.writeResponse(xlpath, sheetName, 16, rowNum, failureReason);
			ExtentManager.logFailDetails("Fail Reason  :"+failureReason);
		}
		catch(Error e) {
			e.printStackTrace();
			failureReason = e.getMessage();
			status = "FAIL";
			ExtentManager.logFailDetails("Test case status :"+status);
			writeReponse.writeResponse(xlpath, sheetName, 15, rowNum, status);
			writeReponse.writeResponse(xlpath, sheetName, 16, rowNum, failureReason);
			ExtentManager.logFailDetails("Fail Reason  :"+failureReason);
		}


		rowNum++;
	}
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
}
package com.ugro.utils;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;



import io.restassured.http.Header;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.markuputils.CodeLanguage;
import com.aventstack.extentreports.markuputils.ExtentColor;
import com.aventstack.extentreports.markuputils.MarkupHelper;
import com.aventstack.extentreports.reporter.ExtentSparkReporter;
import com.aventstack.extentreports.reporter.configuration.Theme;

public class ExtentManager {
	
	public static ExtentReports extentReport;
	public static ExtentReports createInstance(String filename, String reportName, String documentTitle) {
		ExtentSparkReporter extentSparkResport = new ExtentSparkReporter(filename);
		extentSparkResport.config().setReportName(reportName);
		extentSparkResport.config().setDocumentTitle(documentTitle);
		extentSparkResport.config().setTheme(Theme.DARK);
		
		 extentReport = new ExtentReports();
		extentReport.attachReporter(extentSparkResport);
		return extentReport;
		
	}
	
	public static String getReportNamewithTime() {
		
		DateTimeFormatter datetimeformater =  DateTimeFormatter.ofPattern("yyyy_MM_dd_HH_mm_ss");
		LocalDateTime localDatetime = LocalDateTime.now();
		String formattedTime = datetimeformater.format(localDatetime);
		String reportName = "NewReport"+formattedTime+".html";
		
		return reportName;
		
	}
	
	public static void logPassDetails(String log) {
		Setup.extentTest.get().pass(MarkupHelper.createLabel(log, ExtentColor.GREEN));
		
	}
	
	public static void logFailDetails(String log) {
		Setup.extentTest.get().fail(MarkupHelper.createLabel(log, ExtentColor.RED));
		
	}
	
	public static void logInfoDetails(String log) {
		Setup.extentTest.get().info(MarkupHelper.createLabel(log, ExtentColor.GREY));
		
	}
	
	public static void logWarningDetails(String log) {
		Setup.extentTest.get().warning(MarkupHelper.createLabel(log, ExtentColor.ORANGE));
		
	}
	
	public static void logJson(String json) {
		Setup.extentTest.get().info(MarkupHelper.createCodeBlock(json, CodeLanguage.JSON));
		
	}
	
	public static void logHeaders(List<Header> headerList) {
		
		String [][] arrayHeaders = headerList.stream().map(header-> new String[] {header.getName(), header.getValue()})
		.toArray(String [][] :: new);
		Setup.extentTest.get().info(MarkupHelper.createTable(arrayHeaders));
	}

}
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
package com.ugro.utils;

import java.util.HashMap;
import java.util.Map;

public class FailReason{

	Json_Respons_pricing jRP = new Json_Respons_pricing();
	public void failedReason() {

	
		Map<String, String> results = new HashMap<String, String>();
		
		String finalStatus = results.get("finalStatus");
		
		if(finalStatus.equalsIgnoreCase("Pass")) {
			
			System.out.println("Passed");
			
		}
	}

}
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
package com.ugro.utils;

import java.util.List;

import org.json.JSONArray;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;

import io.restassured.response.Response;

public class Json_Respons_pricing {

	public String getresponse = "data.rules_result.application.varResult";
	public String dev = "data.deviation.application";
	public float roi, processingFees, insurencePenitation, cersai, legalTechnicalCharges, documentCharges, loanAmount;

	public String stampDuty, prePaymentCharges, forClouserCharges, applicationProgramCode;
	public String noproertiy;
	public String roideviationCode, ProcessFeedeviationCode1, stampDutyResult, InsurancePendeviationCode2,
			PrePayChargedeviationCode3, ForClosuredeviationCode4, CERSAIdeviationcode,
			legalTechnicalChargesdeviatoncode, documentChargesdeviation;
	public String roiResult, processingFeesResult, insurencePenitationResult, prePaymentChargesResult,
			forClouserChargesResult, cersaiResult, resultstampDuty, legalTechnicalChargesResult, documentChargesResult,
			resultapplicationprogramcode;
	public String final_status;

	public String final_result, deviation_status;
	public String messageroi, message1, response;
	//public String deviationCode;

	public void getJsondata(String response) {
		DocumentContext context = JsonPath.parse(response);

		
		final_status = context.read("data.final_status").toString();
		System.out.println("final status :" + final_status);
		roi = Float.parseFloat(context.read("data.rules_result.application.varResult.standardROI.value").toString());
		processingFees = Float.parseFloat(
				context.read("data.rules_result.application.varResult.standardProcessingFee.value").toString());
		stampDuty = context.read("data.rules_result.application.varResult.stampDuty.value").toString();
		insurencePenitation = Float.parseFloat(
				context.read("data.rules_result.application.varResult.standardInsurancePenetration.value").toString());
		cersai = Float.parseFloat(context.read("data.rules_result.application.varResult.CERSAI.value").toString());
		prePaymentCharges = context.read("data.rules_result.application.varResult.standardPartPaymentcharges.value");
		forClouserCharges = context.read("data.rules_result.application.varResult.standardForeclosureCharges.value")
				.toString();
		noproertiy = context.read("data.output.application.noOfProperties").toString();
		legalTechnicalCharges = Float.parseFloat(
				context.read("data.rules_result.application.varResult.legalTechnicalCharges.value").toString());
		documentCharges = Float
				.parseFloat(context.read("data.rules_result.application.varResult.documentCharges.value").toString());

		applicationProgramCode = context.read("data.output.application.applicationProgramCode").toString();
		loanAmount = Float.parseFloat(context.read("data.output.application.loanAmount").toString());

		roiResult = context.read("data.rules_result.application.varResult.standardROI.result").toString();
		stampDutyResult = context.read("data.rules_result.application.varResult.stampDuty.result").toString();
		processingFeesResult = context.read("data.rules_result.application.varResult.standardProcessingFee.result")
				.toString();
		insurencePenitationResult = context
				.read("data.rules_result.application.varResult.standardInsurancePenetration.result").toString();
		cersaiResult = context.read("data.rules_result.application.varResult.CERSAI.result").toString();
		prePaymentChargesResult = context
				.read("data.rules_result.application.varResult.standardPartPaymentcharges.result").toString();
		forClouserChargesResult = context
				.read("data.rules_result.application.varResult.standardForeclosureCharges.result").toString();
		legalTechnicalChargesResult = context
				.read("data.rules_result.application.varResult.legalTechnicalCharges.result").toString();
		documentChargesResult = context.read("data.rules_result.application.varResult.documentCharges.result")
				.toString();
		// ==================================
				
		if(context.read("data.deviation").toString().length()>2) {
			  roideviationCode =
					  context.read("data.deviation.application.standardROI[0].deviationCode");
					  System.out.println("ROI result is :"+roideviationCode);
					
						 
		} 		
		/*
		 * if(context.read(
		 * "data.deviation.application.standardProcessingFee[0].deviationCode").equals(
		 * "DEV-MICRO-PF-0")) {
		 * 
		 * ProcessFeedeviationCode1 = context.read(
		 * "data.deviation.application.standardProcessingFee[0].deviationCode");
		 * 
		 * InsurancePendeviationCode2 = context.read(
		 * "data.deviation.application.standardInsurancePenetration[0].deviationCode");
		 * PrePayChargedeviationCode3 = context.read(
		 * "data.deviation.application.standardPartPaymentcharges[0].deviationCode");
		 * ForClosuredeviationCode4 = context.read(
		 * "data.deviation.application.standardPartPaymentcharges[0].deviationCode");
		 * CERSAIdeviationcode =
		 * context.read("data.deviation.application.CERSAI[0].deviationCode");
		 * legalTechnicalChargesdeviatoncode =context.read(
		 * "data.deviation.application.legalTechnicalCharges[0].deviationCode");
		 * documentChargesdeviation =
		 * context.read("data.deviation.application.documentCharges[0].deviationCode");
		 * 
		 * }
		 */
		
		
		  //message1 = context.read(dev+ ".standardROI.message").toString();
		final_status = context.read("data.final_status").toString();
		final_result = context.read("data.final_result").toString();
		deviation_status = context.read("data.deviation_status").toString();

	}

}

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
package com.ugro.utils;



	
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.PrintStream;
	import java.util.Properties;

	import org.apache.poi.xssf.usermodel.XSSFCell;
	import org.apache.poi.xssf.usermodel.XSSFRow;
	import org.apache.poi.xssf.usermodel.XSSFSheet;
	import org.apache.poi.xssf.usermodel.XSSFWorkbook;
	import org.json.JSONObject;

	import org.testng.annotations.BeforeClass;

import com.ugro.bre.pricing.test.Ugro_Capital_Test;

import io.restassured.RestAssured;
	import io.restassured.builder.RequestSpecBuilder;
	import io.restassured.filter.log.RequestLoggingFilter;

	import io.restassured.http.ContentType;
import io.restassured.path.json.JsonPath;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;

	public class resonse {

		public String getresponse = "data.rules_result.application.varResult";
		public String dev = "data.deviation.application";
		public float roi, processingFees, insurencePenitation, cersai, legalTechnicalCharges, documentCharges, loanAmount;

		public String stampDuty, prePaymentCharges, forClouserCharges, applicationProgramCode;
		public String noproertiy;
		public String roideviationCode, ProcessFeedeviationCode1, stampDutyResult, InsurancePendeviationCode2,
				PrePayChargedeviationCode3, ForClosuredeviationCode4, CERSAIdeviationcode,
				legalTechnicalChargesdeviatoncode, documentChargesdeviation;
		public String roiResult, processingFeesResult, insurencePenitationResult, prePaymentChargesResult,
				forClouserChargesResult, cersaiResult, resultstampDuty, legalTechnicalChargesResult, documentChargesResult,
				resultapplicationprogramcode;
		public String final_status;

		public String final_result, deviation_status;
		public String messageroi, message1, response;
public Response res;
		public void getJsonreponse(Response res ) throws IOException {
			Ugro_Capital_Test ugr=new Ugro_Capital_Test();
			

			 if(res != null) {
			        JsonPath jsonPath = res.jsonPath();
			        
			        if(jsonPath != null) {
			        	
			        
			        	
			        	
			processingFees = res.jsonPath().getFloat(getresponse + ".standardProcessingFee.value");

			roi = res.jsonPath().getFloat(getresponse + ".standardROI.value");
			// loanAmount=res.jsonPath().getInt("data.output.application.loanAmount");
			insurencePenitation = res.jsonPath().getFloat(getresponse + ".standardInsurancePenetration.value");
			cersai = res.jsonPath().getFloat(getresponse + ".CERSAI.value");
			stampDuty = res.jsonPath().getString(getresponse + ".stampDuty.value");
			prePaymentCharges = res.jsonPath().getString(getresponse + ".standardPartPaymentcharges.value");
			forClouserCharges = res.jsonPath().getString(getresponse + ".standardForeclosureCharges.value");
			// float CERSAI=res.jsonPath().getFloat(getresponse+".");
			noproertiy = res.jsonPath().getString("data.output.application.noOfProperties");
			legalTechnicalCharges = res.jsonPath().getFloat(getresponse + ".legalTechnicalCharges.value");
			documentCharges = res.jsonPath().getFloat(getresponse + ".documentCharges.value");
			 applicationProgramCode = res.jsonPath().getString(
			 "data.output.application.applicationProgramCode");
			 loanAmount = res.jsonPath().getFloat("data.output.application.loanAmount");
			 
			//System.out.println(loanAmount);
			
			//System.out.println(loanAmount);
			//messageroi = res.jsonPath().getString(getresponse + ".standardROI.message");
			
			roiResult = res.jsonPath().getString(getresponse + ".standardROI.result");
			processingFeesResult = res.jsonPath().getString(getresponse + ".standardProcessingFee.result");
			insurencePenitationResult = res.jsonPath().getString(getresponse + ".standardInsurancePenetration.result");
			cersaiResult = res.jsonPath().getString(getresponse + ".CERSAI.result");
			resultstampDuty = res.jsonPath().getString(getresponse + ".stampDuty.result");
			prePaymentChargesResult = res.jsonPath().getString(getresponse + ".standardPartPaymentcharges.result");
			forClouserChargesResult = res.jsonPath().getString(getresponse + ".standardForeclosureCharges.result");
			legalTechnicalChargesResult = res.jsonPath().getString(getresponse + ".legalTechnicalCharges.result");
			documentChargesResult = res.jsonPath().getString(getresponse + ".documentCharges.result");
			resultapplicationprogramcode = res.jsonPath().getString(getresponse + ".applicationProgramCode.result");

			roideviationCode = res.jsonPath().getString("data.deviation.application.standardROI[0].deviationCode");
			ProcessFeedeviationCode1 = res.jsonPath().getString("data.deviation.application.standardProcessingFee[]");
			InsurancePendeviationCode2 = res.jsonPath()
					.getString("data.deviation.application.standardInsurancePenetration[0].deviationCode");
			PrePayChargedeviationCode3 = res.jsonPath().getString("data.deviation.application.standardPartPaymentcharges[0].deviationCode");
			ForClosuredeviationCode4 = res.jsonPath().getString("data.deviation.application.standardForeclosureCharges[0].deviationCode");
			CERSAIdeviationcode = res.jsonPath().getString("data.deviation.application.CERSAI[0].deviationCode");
			legalTechnicalChargesdeviatoncode = res.jsonPath().getString("data.deviation.application.legalTechnicalCharges[0].deviationCode");
			documentChargesdeviation = res.jsonPath().getString("data.deviation.application.documentCharges[0].deviationCode");
			//message1 = res.jsonPath().getString(dev + ".standardROI.message");
//			Assert.assertEquals(applicationProgramCode, "USL-SANJ-ABB");
			
			final_status = res.jsonPath().getString("data.final_status");
			final_result = res.jsonPath().getString("data.final_result");
			deviation_status = res.jsonPath().getString("data.deviation_status");
			
			        } else {
			            System.out.println("JsonPath is null.");
			        }
			    } else {
			        System.out.println("Response is null.");
			    }
		}
}
-----------------------------------------------------------------------------------------------------------------------------------------------
package com.ugro.utils;

import java.util.Arrays;

import org.testng.ITestContext;
import org.testng.ITestListener;
import org.testng.ITestResult;

import com.aventstack.extentreports.ExtentReports;
import com.aventstack.extentreports.ExtentTest;


public class Setup implements ITestListener {
	
	private static ExtentReports extentReport;
	public static ThreadLocal<ExtentTest> extentTest = new ThreadLocal<>(); 
	
	public void onStart(ITestContext context) {
	    
		//Referring to Extent_manager class and there method with name getReportNamewithTime(); 
		
		String fileName = ExtentManager.getReportNamewithTime();
		String fullReportPath = System.getProperty("user.dir")+"\\reports\\"+fileName;
		extentReport = ExtentManager.createInstance(fullReportPath, "Test API Automation", "Test Execution Report");
	  }

	 
	  public void onFinish(ITestContext context) {
	    
		  if(extentReport !=null)
			  extentReport.flush();
		  
	  }
	  
	  public void onTestStart(ITestResult result) {
		    
		 ExtentTest Test =  extentReport.createTest("Test Name "+result.getTestClass().getName()+" - "+ result.getMethod().getMethodName());
		 extentTest.set(Test);
	  }
	  
		
		  public void onTestFailure(ITestResult result) {
		  
		  ExtentManager.logFailDetails(result.getThrowable().getMessage());
		  
		  String excepionMessage = Arrays.toString(result.getThrowable().getStackTrace());
		  extentTest.get()
					.fail("<details>" + "<summary>" + "<b>" + "<font color=" + "red>" + "Exception Occured:Click to see"
							+ "</font>" + "</b >" + "</summary>" + excepionMessage.replaceAll(",", "<br>") + "</details>"
							+ " \n");
		  
		  }
		  }

    -------------------------------------------------------------------------------------------------------------------------------------------------------

    package com.ugro.utils;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import com.ugro.bre.pricing.api.Pricing_Bre;

public class WriteResponse {
	
	static int rowNum =1;

	public void writeResponse(String xlpath, String sheetName, int colNum, int rowNum, String data) {
		try {
			FileInputStream fis = new FileInputStream(xlpath);
			Workbook workbook = new XSSFWorkbook(fis);
			Sheet sheet = workbook.getSheet(sheetName);

			Row row = sheet.getRow(rowNum);
			if (row == null) {
				row = sheet.createRow(rowNum);
			}

			Cell cell = row.getCell(colNum, Row.MissingCellPolicy.CREATE_NULL_AS_BLANK);
			cell.setCellValue(data);

			fis.close();

			FileOutputStream fos = new FileOutputStream(xlpath);
			workbook.write(fos);
			fos.close();

		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	public void validateRsponse() {
		
		
		Pricing_Bre   pricingbrebank = new Pricing_Bre();
		WriteResponse writeReponse = new WriteResponse();
		
		String xlpath = "D:\\Ugro_BRE_Automation\\src\\test\\resources\\data_source\\shakti_sl.xlsx";
		String sheetName = "Sheet1";
		
		
		
		pricingbrebank.callAPI();
		String response = pricingbrebank.callAPI().asString();
		DocumentContext context = JsonPath.parse(response);

		String finalStatus = context.read("data.final_status").toString();
		String roiResult = context.read("data.rules_result.application.varResult.standardROI.result").toString();
		String processingFeesResult = context
				.read("data.rules_result.application.varResult.standardProcessingFee.result").toString();

		if (finalStatus.contains("PASS")) {
			writeReponse.writeResponse(xlpath, sheetName, 13, rowNum, finalStatus);
		} else {

			writeReponse.writeResponse(xlpath, sheetName, 13, rowNum, finalStatus);
			writeReponse.writeResponse(xlpath, sheetName, 14, rowNum, "Fail from roi");
		}
		rowNum++;
	}
}
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
{
    "ruleid": "8777",
    "programName": "Pricing BRE",
    "reference_id": "LAFID294161430",
    "reference_id_2": "",
    "reference_id_3": "",
    "application": {
        "COMMERCIAL": {
            "CERSAI": "${CERSAI_Master}",
            "STAMP_DUTY": "${Stamp_Dutity}",
            "PROCESSING_FEE": "${Processing_Fees}",
            "DOCUMENT_CHARGES": "${Documentation_Charges}",
            "FORECLOSURE_CHARGES": "${Foreclosure}",
            "LEGAL_TECHNICAL_CHARGES": "${Legal_Technical_Charges}",
            "NO_OF_PROPERTIES": "${NO_OF_PROPERTIES}",
            "NET_INSURANCE_PENETRATION": "${Insurance_Penetrat}",
            "PRE_OR_PART_PAYMENT_CHARGES": "${Pre_Part}"
        },
        "LOAN_DETAILS": {
            "ROI": "${ROI}",
            "LOAN_AMOUNT": "${LOAN_AMOUNT}",
            "PROGRAM_CODE": "SL-GROMIC-SHAKTI-GST"
        }
    }
}
--------------------------------------------------------------------------------------------------------------------------------------
pricing_bre_res.json

${ExpectedResponse}
-------------------------------------------------------------------------------------------------------------------------------------
pricing_bre_req.properties

ROI=20
LOAN_AMOUNT=100000
Processing_Fees=1
Insurance_Penetrat=2
CERSAI_Master=400
Stamp_Dutity=no
Pre_Part=no
Foreclosure=no
NO_OF_PROPERTIES=5
Legal_Technical_Charges=2000
Documentation_Charges=1000
ExpectedResponse=xxxx

-----------------------------------------------------------------------------------------------------------------------------
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>
	<groupId>com.zebrunner</groupId>
	<artifactId>carina-demo</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<name>Carina Demo</name>
	<url>http://www.zebrunner.com/</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<carina-core_version>1.2.8</carina-core_version>
		<carina-dataprovider.version>1.2.4</carina-dataprovider.version>
		<carina-api.version>1.2.4</carina-api.version>
		<carina-aws-s3.version>1.2.5</carina-aws-s3.version>
		<carina-azure.version>1.2.5</carina-azure.version>
		<carina-appcenter.version>1.2.7</carina-appcenter.version>

		<java.version>11</java.version>
		<zebrunner-agent.version>1.9.8</zebrunner-agent.version>
		<suite>helloWorld</suite>
	</properties>

	<repositories>
		<repository>
			<id>zebrunner_snapshots</id>
			<name>zebrunner Snapshots</name>
			<url>https://nexus.zebrunner.dev/repository/ce-snapshots/</url>
			<releases>
				<enabled>false</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>

	<dependencies>
		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-core</artifactId>
			<version>${carina-core_version}</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/com.aventstack/extentreports -->
<dependency>
    <groupId>com.aventstack</groupId>
    <artifactId>extentreports</artifactId>
    <version>5.1.1</version>
</dependency>

		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-dataprovider</artifactId>
			<version>${carina-dataprovider.version}</version>
			<exclusions>
				<exclusion>
					<groupId>org.testng</groupId>
					<artifactId>*</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-api</artifactId>
			<version>${carina-api.version}</version>
		</dependency>
		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-aws-s3</artifactId>
			<version>${carina-aws-s3.version}</version>
		</dependency>
		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-azure</artifactId>
			<version>${carina-azure.version}</version>
		</dependency>
		<dependency>
			<groupId>com.zebrunner</groupId>
			<artifactId>carina-appcenter</artifactId>
			<version>${carina-appcenter.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.30</version>
		</dependency>
		<!-- Log4j2 -->
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>2.17.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.17.1</version>
		</dependency>

		<dependency>
			<!--we need it to reuse benefits of zebrunner testng agent for webdriver
				sesssion(s) declaration -->
			<groupId>net.bytebuddy</groupId>
			<artifactId>byte-buddy</artifactId>
			<version>1.14.5</version>
		</dependency>
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.5.6</version>
		</dependency>
		<dependency>
			<groupId>org.postgresql</groupId>
			<artifactId>postgresql</artifactId>
			<version>42.5.3</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.11.0</version>
				<configuration>
					<release>${java.version}</release>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-dependency-plugin</artifactId>
				<version>3.6.0</version>
				<executions>
					<execution>
						<id>copy</id>
						<phase>initialize</phase>
						<goals>
							<goal>copy</goal>
						</goals>
					</execution>
				</executions>
				<configuration>
					<artifactItems>
						<artifactItem>
							<groupId>com.zebrunner</groupId>
							<artifactId>agent-core</artifactId>
							<version>${zebrunner-agent.version}</version>
							<outputDirectory>${project.build.directory}/agent</outputDirectory>
							<destFileName>zebrunner-agent.jar</destFileName>
						</artifactItem>
					</artifactItems>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>3.1.0</version>
				<configuration>
					<argLine>-javaagent:${project.build.directory}/agent/zebrunner-agent.jar</argLine>
					<useSystemClassLoader>false</useSystemClassLoader>
					<suiteXmlFiles>
						<suiteXmlFile>${project.build.directory}/test-classes/testng_suites/${suite}.xml</suiteXmlFile>
					</suiteXmlFiles>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>

--------------------------------------------------------------------------------------------------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Suite">
<listeners>
	<listener class-name = "com.ugro.utils.Setup"/>
	
</listeners>
  <test thread-count="5" name="Test">
    <classes>
      <class name="com.ugro.bre.pricing.test.Ugro_Capital_Test"/>
    </classes>
  </test> <!-- Test -->
</suite> <!-- Suite -->
