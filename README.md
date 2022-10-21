# GiuaKyPhanTan

============ pom.xml =========
	<dependencies>
		<dependency>
			<groupId>org.mongodb</groupId>
			<artifactId>mongodb-driver-sync</artifactId>
			<version>4.7.1</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.mongodb/mongo-java-driver -->
		<dependency>
			<groupId>org.mongodb</groupId>
			<artifactId>mongo-java-driver</artifactId>
			<version>3.12.11</version>
		</dependency>
	</dependencies>
	
	<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>2.3.1</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
			</configuration>
		</plugin>
	        <plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-eclipse-plugin</artifactId>
			<configuration>
				<downloadSources>true</downloadSources>
				<downloadJavadocs>true</downloadJavadocs>
			</configuration>
		</plugin>

	</plugins>
  </build>
  
  ================================ MongoConnection.java ======================
  package untils;

import com.mongodb.ConnectionString;
import com.mongodb.MongoClientSettings;
import com.mongodb.ServerApi;
import com.mongodb.ServerApiVersion;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;

public class MongoConnection {
	private static MongoConnection instance;
    private MongoClient mongoClient;

    private MongoConnection() {
    	ConnectionString connectionString = new ConnectionString(
				"mongodb+srv://VoThiTraGiang_19510771:giangvo19510771@cluster0.1odnozv.mongodb.net/?retryWrites=true&w=majority");
		MongoClientSettings settings = MongoClientSettings.builder().applyConnectionString(connectionString)
				.serverApi(ServerApi.builder().version(ServerApiVersion.V1).build()).build();
		mongoClient = MongoClients.create(settings);
    }

    public synchronized static MongoConnection getInstance() {
        if(instance == null)
            instance = new MongoConnection();

        return instance;
    }

    public MongoClient getMongoClient() {
        return mongoClient;
    }
}
=======-======================= ENTITY POJO ===============
package entity;

import java.time.LocalDate;
import java.util.List;

import org.bson.BsonType;
import org.bson.codecs.pojo.annotations.BsonId;
import org.bson.codecs.pojo.annotations.BsonProperty;
import org.bson.codecs.pojo.annotations.BsonRepresentation;

public class Customer {
	@BsonId
	@BsonProperty("_id")
	private String customerId;
	
	private String email;
	@BsonProperty("first_name")
	private String firstName;
	@BsonProperty("last_name")
	private String lastName;
	@BsonProperty("registration_date")
	private LocalDate registrationDate;
	private Address address;
	private List<Phone> phones;

	public Customer() {
	}

	public Customer(String customerId, String email, String firstName, String lastName, LocalDate registrationDate,
			Address address, List<Phone> phones) {
		this.customerId = customerId;
		this.email = email;
		this.firstName = firstName;
		this.lastName = lastName;
		this.registrationDate = registrationDate;
		this.address = address;
		this.phones = phones;
	}

	public Customer(String customerId, String email, String firstName, String lastName, LocalDate registrationDate,
			Address address) {
		this.customerId = customerId;
		this.email = email;
		this.firstName = firstName;
		this.lastName = lastName;
		this.registrationDate = registrationDate;
		this.address = address;
	}

	public String getCustomerId() {
		return customerId;
	}

	public void setCustomerId(String customerId) {
		this.customerId = customerId;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public String getFirstName() {
		return firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public LocalDate getRegistrationDate() {
		return registrationDate;
	}

	public void setRegistrationDate(LocalDate registrationDate) {
		this.registrationDate = registrationDate;
	}

	public Address getAddress() {
		return address;
	}

	public void setAddress(Address address) {
		this.address = address;
	}

	public List<Phone> getPhones() {
		return phones;
	}

	public void setPhones(List<Phone> phones) {
		this.phones = phones;
	}

	@Override
	public String toString() {
		return "Customer{" + "customerId='" + customerId + '\'' + ", email='" + email + '\'' + ", firstName='"
				+ firstName + '\'' + ", lastName='" + lastName + '\'' + ", registrationDate=" + registrationDate
				+ ", address=" + address + ", phones=" + phones + '}';
	}
}
======================== ENTITY DOCUMENT 
package Entity;

import java.util.Date;
import java.util.List;

public class Order {
	private  String _id, status;
	private Date orderDate, shippedDate;
	private Customer customer;
	private Staff staff;
	private double total;
	private Address shippingAddress;
	public List<OrderDetail> orderDetail;
	public Order() {
		// TODO Auto-generated constructor stub
	}

}
/===========
public class OrderDetail {
	private int quality;
	private Product product;
	private double lineTotal, price, discount;
	private List<String> colors;

	public OrderDetail() {
		// TODO Auto-generated constructor stub
	}
============================ DAO ====================
==== DAO POJO ============
package dao;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Map;

import org.bson.BsonNull;
import org.bson.Document;
import org.bson.codecs.configuration.CodecRegistries;
import org.bson.codecs.configuration.CodecRegistry;
import org.bson.codecs.pojo.PojoCodecProvider;
import org.bson.conversions.Bson;

import com.mongodb.MongoClientSettings;
import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;

import entity.Customer;
import entity.Phone;

public class CustomerDao {
	MongoCollection<Customer> collection;

	public CustomerDao(MongoClient client, String dbName) {
		// TODO Auto-generated constructor stub
		PojoCodecProvider codecProvider = PojoCodecProvider.builder().automatic(true).build();

		CodecRegistry codecRegistry = CodecRegistries.fromRegistries(MongoClientSettings.getDefaultCodecRegistry(),
				CodecRegistries.fromProviders(codecProvider));

		collection = client.getDatabase(dbName).getCollection("customers", Customer.class)
				.withCodecRegistry(codecRegistry);
	}

	public List<Customer> getAll() {
		List<Customer> customers = new ArrayList<Customer>();
		return collection.find().into(customers);
	}

	public boolean insertOne(Customer customer) {
		collection.insertOne(customer);
		try {
			Thread.sleep(20);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("Insert finish");
		return true;
	}

	public boolean insertManyCustomer(List<Customer> customers) {
		collection.insertMany(customers);
		try {
			Thread.sleep(20);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("Insert many finish");
		return true;
	}

	public Customer getCustomerById(String id) {
		return collection.find(Filters.eq("_id", id)).first();
	}

	public boolean updateOnecustomer(String id) {
		List<Phone> phones = new ArrayList<Phone>();
		phones.add(new Phone("AAA", "1234567"));
//		 Updates.unset(id) ===  xoa filed
		Bson updates = Updates.combine(Updates.set("last_name", "Con Con"), Updates.pushEach("phones", phones),
				Updates.addToSet("lover", "Giang"), Updates.currentTimestamp("lastUpdated"));
		collection.updateOne(Filters.eq("_id", id), updates);

		try {
			Thread.sleep(20);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("update finish");
		return true;
	}

	public boolean updateMany(List<Customer> customers) {

		return true;
	}

	public boolean deleteOneCustomer(String id) {
		collection.deleteOne(Filters.eq("_id", id));
		try {
			Thread.sleep(20);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("delete finish");
		return true;
	}

	public boolean addField(String id) {
		collection.aggregate(Arrays.asList(new Document("$match", new Document("_id", "GIANG3")),
				new Document("$addFields", new Document("lover", "Giang"))));
		try {
			Thread.sleep(20);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		System.out.println("add field finish");
		return true;
	}

	public Map<String, Integer> getNumberCustomerByState(String state) {
		Map<String, Integer> lst = null;
		collection.aggregate(Arrays.asList(new Document("$match", 
			    new Document("address.state", "NY")))).forEach(e -> {
			    	
			    });
//		collection.aggregate(Arrays.asList(new Document("$match", 
//			    new Document("address.state", "NY")), 
//			    new Document("$project", 
//			    new Document("_id", 0L)
//			            .append("address.state", 1L)), 
//			    new Document("$group", 
//			    new Document("_id", 
//			    new BsonNull())
//			            .append("count", 
//			    new Document("$sum", 1L))), 
//			    new Document("$project", 
//			    new Document("_id", 0L)
//			            .append("count", 1L)))).forEach(e -> System.out.println(e));
//
//		
		return lst;
	}

}
====== DAO DOCUMENT =======


package Execise_2.dao;

import static com.mongodb.client.model.Filters.eq;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

import javax.print.Doc;

import org.bson.BsonObjectId;
import org.bson.Document;
import org.bson.conversions.Bson;
import org.bson.types.ObjectId;

import com.mongodb.client.AggregateIterable;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.model.Filters;
import com.mongodb.client.model.Updates;
import com.mongodb.client.result.InsertOneResult;

import Execise_2.busimpl.Mapper;
import Execise_2.entity.Zip;
import Utils.MongoClientConnectApplication;

public class ZipDao {
	private MongoCollection<Document> zipCol;

	public ZipDao() {
		// TODO Auto-generated constructor stub
		zipCol = MongoClientConnectApplication.getInstance().getMongoClient().getDatabase("sample_training")
				.getCollection("zips");
	}

	// Hiển thị n documents từ document thứ k
	public AggregateIterable<Document> showDocumentFromIndex(int n, int k) {
		return zipCol.aggregate(Arrays.asList(new Document("$skip", k - 1), new Document("$limit", n + 1)));
	}

	public Zip getDocumentByID(String id) {
		Document document = zipCol.find(Filters.eq("_id", new ObjectId(id))).first();
		return Mapper.exportToZip(document);
	}

	public String insertNewZip(Zip zip) {
		Document item = new Document(new Document("city", zip.getCity()).append("zip", zip.getZips())
				.append("loc", new Document("y", zip.getLocation().getX()).append("x", zip.getLocation().getY()))
				.append("pop", zip.getPop()).append("state", zip.getState()));
		InsertOneResult result = zipCol.insertOne(item);
		if (result.getInsertedId().isNull())
			return null;
		return item.getObjectId("_id").toHexString();
	}

//	Cập nhật thông tin của một document khi biết id
	public boolean updateDocumentByID(Zip zip, String id) {
		try {
			Document loc = new Document().append("x", zip.getLocation().getX()).append("y", zip.getLocation().getY());
			Bson updates = Updates.combine(Updates.set("city", zip.getCity()), Updates.set("state", zip.getState()),
					Updates.set("zips", zip.getState()), Updates.set("pop", zip.getState()), Updates.set("loc", loc),
					Updates.currentTimestamp("lastUpdated"));
			zipCol.updateOne(Filters.eq("_id", new ObjectId(id)), updates);

		} catch (Exception e) {
			// TODO: handle exception
			System.out.println(e.getMessage());
			return false;
		}
		return true;
	}

//	Tìm các document có city là PALMER
	public List<Document> getZipByCity(String city) {

		return zipCol.find(eq("city", city)).into(new ArrayList<>());
	}
	
//	Tìm các document có dân số >100000
//	db.zips.find({pop:{$gt:100000}})
	public List<Document> getZipByPop(int pop) {
		
		Document filter = Document.parse("{pop:{$gt:"+ pop +"}}");		
		return zipCol.find(filter).into(new ArrayList<>());
	}

	public int getPopOfZipByNameCity(String name)
	{
		return (int)zipCol.find(Filters.eq("city", name)).first().get("pop");
	}
//	Tìm các thành phố có dân số từ 10 – 50
	public List<Document> getCityByPop(int popF, int popT){
		Document filter = Document.parse("{pop:{$gt:"+ popF +",$lt:"+ popT +"}}");
		return zipCol.find(filter).into(new ArrayList<>());
	}
	
//	Tìm tất cả các thành phố của bang MA có dân số trên 500
	public List<Document> getAllCityOgMAHavePop(){
		Document filter = Document.parse("{{state: \"MA\",pop: {$gt:500}}}");
		return zipCol.find(filter).into(new ArrayList<>());
	}
	
}
=================== MAPPER DOCUMENT =================
public class Mapper {

	public static Zip exportToZip(Document doc) {

		Document locationDoc = (Document) doc.get("loc");
		Location location = new Location(locationDoc.getDouble("x"), locationDoc.getDouble("y"));

		return new Zip(doc.getObjectId("_id").toHexString(), doc.getString("city"), doc.getString("state"),
				doc.getString("zips"), doc.getInteger("pop", 0), location);
	}

	public static Document transformToBson(Zip zip) {
		return new Document(new Document("city", zip.getCity())
				.append("zip", zip.getZips())
				.append("_id", zip.getId())
				.append("loc", new Document("y", zip.getLocation().getX()).append("x", zip.getLocation().getY()))
				.append("pop", zip.getPop()).append("state", zip.getState()));
	}

}

=============== APPLICATION
public class Application {
	
	public static String DB_NAME = "BikeStores";
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		MongoClient connection = MongoConnection.getInstance().getMongoClient();
		
//		ProductDao proDao = new ProductDao(connection, DB_NAME);
//		proDao.getAll().forEach(a -> System.out.println(a));
		
		CustomerDao customerDao = new CustomerDao(connection, DB_NAME);
//		customerDao.getAll().forEach(e-> System.out.println(e));
		
		// addOnne Customer
//		Customer cus = new Customer("GIANG2", "giang@gmail.com", "Canhcut", "Con", LocalDate.now(), 
//				new Address("TP HCM", "AL", "NguyenVanBao", 110000),null);
//		List<Customer> customers = new ArrayList<Customer>();
//		customers.add(new Customer("GIANG3", "giang@gmail.com", "Canhcut", "Con", LocalDate.now(), 
//				new Address("TP HCM", "AL", "NguyenVanBao", 110000),null));
//		customers.add(new Customer("GIANG4", "giang@gmail.com", "Canhcut", "Con", LocalDate.now(), 
//				new Address("TP HCM", "AL", "NguyenVanBao", 110000),null));
//		customers.add(new Customer("GIANG5", "giang@gmail.com", "Canhcut", "Con", LocalDate.now(), 
//				new Address("TP HCM", "AL", "NguyenVanBao", 110000),null));
//		customerDao.insertManyCustomer(customers);
		//update one
//		customerDao.updateOnecustomer(new Customer("GIANG2", "giang@gmail.com", "Canhcut", "Con", LocalDate.now(), 
//				new Address("TP HCM", "AL", "NguyenVanBao", 110000),null));
		//delete one
//		customerDao.deleteOneCustomer("GIANG2");
//		customerDao.addField("GENO9");
//		customerDao.updateOnecustomer("GENO9");
//		System.out.println(customerDao.getCustomerById("GENO9"));
		
		customerDao.getNumberCustomerByState("NY");
	}

}

  
