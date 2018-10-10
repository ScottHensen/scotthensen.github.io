---
layout: post
title:  "JPA"
date:   2018-10-09 12:00:00 -0700
categories: java spring data jpa
---
# Spring Data JPA - a dumb way to negate your lazy loading
I was playing with different options for joining tables, and kept scratching my
head about why JPA was building so many dang SQL selects.  Eventually, the
lightbulb came on.  I was logging my result sets so that I could verify I was
getting all of my data.  Lazy loading promises to not select on joined tables
*until you make a reference to them*.  Since I was referring to them in my
Entity's toString() method, I was accidentally cancelling out my lazy loading.  

Derp.  

Here's the code...  

**Pet Owners @Controller**  
```java
@RequestMapping( { "", "/", "/index", "/index.html" } )
public String listOwners(Model model)
{
	log.info("\n >>>>> ownerSvc.findAll()");
	Set<Owner> allOwners = ownerSvc.findAll();
	log.info(allOwners.toString());
	model.addAttribute( "owners", allOwners );

	log.info("\n >>>>> ownerSvc.findByLastName()");
	Owner owner = ownerSvc.findByLastName("Doolittle");
	log.info(owner.toString());
	model.addAttribute( "ownerEntity",  owner);

	log.info("\n >>>>> ownerSvc.findOwnerPetsByLastName()");
	List<OwnerPetsDTO> ownerPets = ownerSvc.findOwnerPetsByLastName("Doolittle");
	log.info(ownerPets.toString());
	model.addAttribute( "ownerPetsDTO", ownerPets);

	return "owners/index";
}
```  
**Owner @Service Impl**  
```java
@Override
public Set<Owner> findAll()
{
	return ownerRepo.findAll();
}

@Override
public Owner findByLastName(String lastName)
{
	return ownerRepo.findByLastName(lastName);
}

@Override
public List<OwnerPetsDTO> findOwnerPetsByLastName(String lastName)
{
	return ownerRepo.findOwnerPetsByLastName(lastName);
}
```  
**Owner Repository**  
```java
@Repository
public interface OwnerRepository extends CrudRepository<Owner, Long>
{
	Owner findByLastName(String lastName);

	Set<Owner> findAll();

	@Query( value = "SELECT new com.scotthensen.fooclinic.model.OwnerPetsDTO "
				  + "     ( o.firstName,   "
				  + "       o.lastName,    "
				  + "       o.address,     "
				  + "       p.name       ) "
				  + "FROM   Owner o,       "
				  + "       Pet p          "
				  + "WHERE  o.lastName = :lastName  "
				  + "  AND  o.id       = p.owner    "
			  )
	List<OwnerPetsDTO> findOwnerPetsByLastName(@Param("lastName") String lastName);
}
```
**Owner @Entity**  
```java
@Getter
@Setter
@NoArgsConstructor
@Entity
@Table( name = "owners" )
public class Owner extends Person
{
	@Builder
	public Owner(Long id, String firstName, String lastName,
				 String address, String city, String telephone, Set<Pet> pets)
	{
		super(id, firstName, lastName);
		this.address = address;
		this.city = city;
		this.telephone = telephone;
		this.pets = pets;
	}

	@Column( name = "address" )
	private String address;

	@Column( name = "city" )
	private String city;

	@Column( name = "telephone" )
	private String telephone;

	@OneToMany( cascade = CascadeType.ALL, mappedBy = "owner", fetch = FetchType.LAZY )
	private Set<Pet> pets = new HashSet<>();

	@Override
	public String toString() {
		return "Owner [ \n"
				+ "      address="   + address   + "\n"
				+ "    , city="      + city      + "\n"
				+ "    , telephone=" + telephone + "\n"
				+ "    , pets="      + pets      + "\n"	 // this will negate lazy loading
				+ "]";
	}
}
```  
**Pet @Entity**  
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Entity
@Table( name = "pets" )
public class Pet extends BaseEntity
{
	@Column( name = "name")
	private String  name;

	@ManyToOne( fetch = FetchType.LAZY )
	@JoinColumn( name = "type_id" )
	private PetType petType;

	@ManyToOne( fetch = FetchType.LAZY )
	@JoinColumn( name = "owner_id" )
	private Owner   owner;

	@Column( name = "birth_date")
	private LocalDate birthDate;

	@OneToMany( cascade = CascadeType.ALL, mappedBy = "pet", fetch = FetchType.LAZY )
	private Set<Visit> visits = new HashSet<>();

	@Override
	public String toString() {
		return "Pet ["
				+ "       name="      + name      + "\n"
				+ "     , petType="   + petType   + "\n"	// this will negate lazy loading
				+ "     , birthDate=" + birthDate + "\n"
				+ "     , visits="    + visits    + "\n"	// this will negate lazy loading
				+ "]";
	}
}
```  
**Owner Pets DTO**
```java  
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class OwnerPetsDTO
{
	private String firstName;
	private String lastName;
	private String address;
	private String petName;

	@Override
	public String toString() {
		return "OwnerPetsDTO [ \n"
			+  "    firstName=" + firstName + "\n"
			+  "  , lastName="  + lastName  + "\n"
			+  "  , address="   + address   + "\n"
			+  "  , petName="   + petName   + "\n"
			+  "]";
	}
}
```   
**Results (lazy loading, but logging the relationships with toString())**  


**findAll**
```sql
>>>>> ownerSvc.findAll()
Hibernate:
	 select
			 owner0_.id as id1_0_,
			 owner0_.first_name as first_na2_0_,
			 owner0_.last_name as last_nam3_0_,
			 owner0_.address as address4_0_,
			 owner0_.city as city5_0_,
			 owner0_.telephone as telephon6_0_
	 from
			 owners owner0_
Hibernate:
	 select
			 pets0_.owner_id as owner_id4_1_0_,
			 pets0_.id as id1_1_0_,
			 pets0_.id as id1_1_1_,
			 pets0_.birth_date as birth_da2_1_1_,
			 pets0_.name as name3_1_1_,
			 pets0_.owner_id as owner_id4_1_1_,
			 pets0_.type_id as type_id5_1_1_
	 from
			 pets pets0_
	 where
			 pets0_.owner_id=?
Hibernate:
	 select
			 pettype0_.id as id1_3_0_,
			 pettype0_.name as name2_3_0_
	 from
			 types pettype0_
	 where
			 pettype0_.id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 pets0_.owner_id as owner_id4_1_0_,
			 pets0_.id as id1_1_0_,
			 pets0_.id as id1_1_1_,
			 pets0_.birth_date as birth_da2_1_1_,
			 pets0_.name as name3_1_1_,
			 pets0_.owner_id as owner_id4_1_1_,
			 pets0_.type_id as type_id5_1_1_
	 from
			 pets pets0_
	 where
			 pets0_.owner_id=?
Hibernate:
	 select
			 pettype0_.id as id1_3_0_,
			 pettype0_.name as name2_3_0_
	 from
			 types pettype0_
	 where
			 pettype0_.id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 pets0_.owner_id as owner_id4_1_0_,
			 pets0_.id as id1_1_0_,
			 pets0_.id as id1_1_1_,
			 pets0_.birth_date as birth_da2_1_1_,
			 pets0_.name as name3_1_1_,
			 pets0_.owner_id as owner_id4_1_1_,
			 pets0_.type_id as type_id5_1_1_
	 from
			 pets pets0_
	 where
			 pets0_.owner_id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
Hibernate:
	 select
			 visits0_.pet_id as pet_id4_6_0_,
			 visits0_.id as id1_6_0_,
			 visits0_.id as id1_6_1_,
			 visits0_.date as date2_6_1_,
			 visits0_.description as descript3_6_1_,
			 visits0_.pet_id as pet_id4_6_1_
	 from
			 visits visits0_
	 where
			 visits0_.pet_id=?
```   
```text
[Owner [
		 address=123 Foo St
	 , city=Tempe
	 , telephone=4808675309
	 , pets=[Pet [       name=Pooch
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=2018-10-09
		, visits=[]
]]
], Owner [
		 address=123 Foo St
	 , city=Tempe
	 , telephone=4808675309
	 , pets=[Pet [       name=Kitty
		, petType=com.scotthensen.fooclinic.model.PetType@6f40601d
		, birthDate=2018-10-09
		, visits=[com.scotthensen.fooclinic.model.Visit@2acaf15a]
]]
], Owner [
		 address=1313 Mockingbird Ln
	 , city=Tempe
	 , telephone=4808675309
	 , pets=[Pet [       name=dog2
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog3
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog4
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog1
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog5
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
]]
]]
```  
**findByLastName**  
```sql
>>>>> ownerSvc.findByLastName()
Hibernate:
	 select
			 owner0_.id as id1_0_,
			 owner0_.first_name as first_na2_0_,
			 owner0_.last_name as last_nam3_0_,
			 owner0_.address as address4_0_,
			 owner0_.city as city5_0_,
			 owner0_.telephone as telephon6_0_
	 from
			 owners owner0_
	 where
			 owner0_.last_name=?
```
```text
Owner [
		 address=1313 Mockingbird Ln
	 , city=Tempe
	 , telephone=4808675309
	 , pets=[Pet [       name=dog2
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog3
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog4
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog1
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
], Pet [       name=dog5
		, petType=com.scotthensen.fooclinic.model.PetType@6df38ecb
		, birthDate=null
		, visits=[]
]]
]
```  
**findOwnerPetsByLastName**  
```sql
>>>>> ownerSvc.findOwnerPetsByLastName()
Hibernate:
	 select
			 owner0_.first_name as col_0_0_,
			 owner0_.last_name as col_1_0_,
			 owner0_.address as col_2_0_,
			 pet1_.name as col_3_0_
	 from
			 owners owner0_ cross
	 join
			 pets pet1_
	 where
			 owner0_.last_name=?
			 and owner0_.id=pet1_.owner_id
```
```text
[OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog2
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog5
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog1
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog4
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog3
]]
```  
## Fixing the Lazy Loading
**Removed petType and visits from Pet toString()**    
```java
@Override
public String toString() {
	return    "Pet ["
      + "       name="      + name      + "\n"
//    + "     , petType="   + petType   + "\n"	// this will negate lazy loading
      + "     , birthDate=" + birthDate + "\n"
//    + "     , visits="    + visits    + "\n"	// this will negate lazy loading
      + "]";
}
```  
**Removed pets from Owner toString()**  
```java
@Override
public String toString() {
	return "Owner [ \n"
			+ "      address="   + address   + "\n"
			+ "    , city="      + city      + "\n"
			+ "    , telephone=" + telephone + "\n"
//		+ "    , pets="      + pets      + "\n"	 // this will negate lazy loading
			+ "]";
}
```
**Results (Lazy loading works when you don't log your related tables.)**  


**findAll**
```sql
>>>>> ownerSvc.findAll()
Hibernate:
	 select
			 owner0_.id as id1_0_,
			 owner0_.first_name as first_na2_0_,
			 owner0_.last_name as last_nam3_0_,
			 owner0_.address as address4_0_,
			 owner0_.city as city5_0_,
			 owner0_.telephone as telephon6_0_
	 from
			 owners owner0_
```
```text
[Owner [
     address=123 Foo St
   , city=Tempe
   , telephone=4808675309
], Owner [
     address=123 Foo St
   , city=Tempe
   , telephone=4808675309
], Owner [
		 address=1313 Mockingbird Ln
	 , city=Tempe
	 , telephone=4808675309
]]
```
**findByLastName**  
```sql
>>>>> ownerSvc.findByLastName()
Hibernate:
	 select
			 owner0_.id as id1_0_,
			 owner0_.first_name as first_na2_0_,
			 owner0_.last_name as last_nam3_0_,
			 owner0_.address as address4_0_,
			 owner0_.city as city5_0_,
			 owner0_.telephone as telephon6_0_
	 from
			 owners owner0_
	 where
			 owner0_.last_name=?
```
```text
Owner [
		 address=1313 Mockingbird Ln
	 , city=Tempe
	 , telephone=4808675309
]
```  
**findOwnerPetsByLastName**  
```sql
>>>>> ownerSvc.findOwnerPetsByLastName()
Hibernate:
	 select
			 owner0_.first_name as col_0_0_,
			 owner0_.last_name as col_1_0_,
			 owner0_.address as col_2_0_,
			 pet1_.name as col_3_0_
	 from
			 owners owner0_ cross
	 join
			 pets pet1_
	 where
			 owner0_.last_name=?
			 and owner0_.id=pet1_.owner_id
```
```text
[OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog1
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog4
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog5
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog2
], OwnerPetsDTO [
	 firstName=Dr.
 , lastName=Doolittle
 , address=1313 Mockingbird Ln
 , petName=dog3
]]
```  
