# How to Create a REST API With Spring Boot？

1. 创建spring boot项目
2. 完成相关配置，如数据库配置等
3. 创建一个实体
4. 编写controller,service,dao层
5. 测试

## 1 创建spring boot项目

spring boot + mongodb

### 1.1 引入常用依赖

```java
		<dependency>
			<!-- 热部署依赖 -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
		</dependency>
 		<dependency>
		<!-- lombook -->
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>provided</scope>
		</dependency>
 		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>
```

## 2 完成相关配置，如数据库配置等

```java
#普通属性值配置
server.port=8080
server.servlet.context-path=/reporting
#mongdb配置
spring.data.mongodb.uri=mongodb://localhost:27017/test

```

## 3 创建一个实体

```java
package com.reporting.entity;

import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.stereotype.Component;

import lombok.Data;


@Data
@Component	
@Document(collection = "reporting")
public class DiagnoseReport implements Serializable{
	@Id
	private String id;
	private String patientMail;
	private String doctorMail;
	private String description;
	private String medicine;
	private String reportTime;
}

```

## 4 编写controller,service,dao层

### 4.1 controller

```java
package com.reporting.controller;

import java.util.Date;
import java.util.List;
import java.util.Optional;

import org.omg.PortableServer.IdAssignmentPolicy;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.reporting.entity.DiagnoseReport;
import com.reporting.service.ReportService;

import lombok.Data;

@RestController
@RequestMapping("/users")
public class ReportController {

	@Autowired
	private ReportService reportService;

	@GetMapping("/mail={mail}")
	public List<DiagnoseReport> getReportByPatientName(@PathVariable String mail) {
		List<DiagnoseReport> diagnoseReports = reportService.getReportByPatientName(mail);
		System.out.println(diagnoseReports);
		return diagnoseReports;
	}

	@GetMapping("/id={id}")
	public Optional<DiagnoseReport> getReportById(@PathVariable String id) {
		return reportService.getReportById(id);
	}

	@GetMapping("/all")
	public List<DiagnoseReport> getAllReports() {
		return reportService.findAllDiagnoseReports();
	}

	@GetMapping("/update/id={id}")
	public boolean updateReport(@PathVariable String id) {
		DiagnoseReport diagnoseReport = new DiagnoseReport();
		diagnoseReport.setId(id);
		diagnoseReport.setDescription("123456");
		diagnoseReport.setDoctorMail("123@qq.com");
		diagnoseReport.setMedicine("123");
		diagnoseReport.setPatientMail("456@qq.com");
		Date date = new Date();
		diagnoseReport.setReportTime(date.toString());
		return reportService.updateReport(diagnoseReport);
	}
	
	@GetMapping("/delete/id={id}")
	public boolean deleteReportById(@PathVariable String id) {
		return reportService.deleteById(id);
	}
	
	@GetMapping("add")
	public boolean addReport() {
		DiagnoseReport diagnoseReport = new DiagnoseReport();
		diagnoseReport.setDescription("描述");
		diagnoseReport.setDoctorMail("11111@163.com");
		diagnoseReport.setMedicine("药物");
		diagnoseReport.setPatientMail("aaaaaa@qq.com");
		Date date = new Date();
		diagnoseReport.setReportTime(date.toString());
		return reportService.addReport(diagnoseReport);
	}

}

```

### 4.2 service

```java
package com.reporting.service;

import java.util.List;
import java.util.Optional;

import org.bson.Document;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.stereotype.Service;

import com.reporting.dao.ReportDao;
import com.reporting.entity.DiagnoseReport;

@Service
public class ReportService {
	@Autowired
	private ReportDao reportDao;
	@Autowired
	private MongoTemplate mongoTemplate;

	public List<DiagnoseReport> getReportByPatientName(String mail) {
		// 创建条件对象
		Criteria criteria = Criteria.where("patientMail").is(mail);
		// 创建查询对象，然后将条件对象添加到其中
		Query query = new Query(criteria);
		// 查询并返回结果
		List<DiagnoseReport> reportList = mongoTemplate.find(query, DiagnoseReport.class);

		return reportList;

	}

	public Optional<DiagnoseReport> getReportById(String id) {
		return reportDao.findById(id);
	}

	public List<DiagnoseReport> findAllDiagnoseReports() {
		return reportDao.findAll();
	}

	public boolean updateReport(DiagnoseReport report) {
		try {
			Query query = new Query(Criteria.where("id").is(report.getId()));
			Document doc = new Document(); // org.bson.Document
			mongoTemplate.getConverter().write(report, doc);
			Update update = Update.fromDocument(doc);
//			mongoTemplate.upsert(query, update, "report");
			mongoTemplate.updateFirst(query, update, DiagnoseReport.class);
		} catch (Exception e) {
			// TODO: handle exception
			System.out.println(e);
			return false;
		}
		return true;
	}

	public boolean deleteById(String id) {
		try {
			Criteria criteria = Criteria.where("id").is(id);
			Query query = new Query(criteria);
			// 执行删除查找到的匹配的第一条文档,并返回删除的文档信息
			mongoTemplate.findAndRemove(query, DiagnoseReport.class);
		} catch (Exception e) {
			// TODO: handle exception
			System.out.println(e);
			return false;
		}
		return true;
	}
	
	public boolean addReport(DiagnoseReport report) {
		try {
			mongoTemplate.save(report);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			return false;
		}
		return true;
	}
}

```

### 4.3 dao

```java
package com.reporting.dao;

import org.springframework.data.mongodb.repository.MongoRepository;

import com.reporting.entity.DiagnoseReport;

public interface ReportDao extends MongoRepository<DiagnoseReport, String>{
	
}

```

## 5 测试

### 5.1 使用RestTemplate测试暴露在外的rest API接口

一般情况下有如下三种http客户端工具类包都可以方便的进行http服务调用:

- httpClient
- okHttp
- JDK原生URLConnection

spring提供了RestTemplate的工具类对上述的3种http客户端工具类进行了封装，可在spring项目中使用RestTemplate进行服务调用。|

```java
package com.reporting;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.ResponseEntity;
import org.springframework.web.client.RestTemplate;

import com.reporting.entity.DiagnoseReport;

//调用rest API进行增删改查操作

@SpringBootTest
public class RestTemplateTest {
	@Autowired
	private RestTemplate restTemplate;

	@Test
	public void findById() {
		// 返回一个对象
		String url = "http://localhost:8080/reporting/users/id=0";
		// restTemplate可以对json格式字符串反序列化
		ResponseEntity<DiagnoseReport> report = restTemplate.getForEntity(url, DiagnoseReport.class);
		System.out.println(report);
		DiagnoseReport objects = report.getBody();
		System.out.println(objects);
	}

	@Test
	public void findAll() {
		// 返回一个对象列表
		String url = "http://localhost:8080/reporting/users/all";
		ResponseEntity<DiagnoseReport[]> forEntity = restTemplate.getForEntity(url, DiagnoseReport[].class);
		DiagnoseReport[] body = forEntity.getBody();
		for (DiagnoseReport diagnoseReport : body) {
			System.out.println(diagnoseReport);
		}
	}

	@Test
	public void update() {
		// 返回一个对象列表
		String url = "http://localhost:8080/reporting/users/update/id=5f8258b4e609c76b611634b6";
		// 查找该id对应对象是否存在
		ResponseEntity<Boolean> forEntity = restTemplate.getForEntity(url, Boolean.class);
		Boolean body = forEntity.getBody();
		if (body == true) {
			System.out.println("更新成功");
		} else {
			System.out.println("更新失败");
		}
	}

	@Test
	public void delete() {
		// 返回一个对象列表
		String url = "http://localhost:8080/reporting/users/delete/id=5f6aaa218a379a38a31ec2e8";
		// 查找该id对应对象是否存在
		ResponseEntity<Boolean> forEntity = restTemplate.getForEntity(url, Boolean.class);
		Boolean body = forEntity.getBody();
		if (body == true) {
			System.out.println("删除成功");
		} else {
			System.out.println("删除失败");
		}
	}
	@Test
	public void add() {
		String url = "http://localhost:8080/reporting/users/add";
		ResponseEntity<Boolean> forEntity = restTemplate.getForEntity(url, Boolean.class);	
		Boolean body = forEntity.getBody();
		if (body == true) {
			System.out.println("增加成功");
		} else {
			System.out.println("增加失败");
		}
	}

}

```

### 5.2 结果

测试结果

![image-20201015114216984](spring boot/image/image-20201015114216984.png)

浏览器访问

![image-20201015114552327](spring boot/image/image-20201015114244553.png)

