---
title: "Xml"
date: 2021-11-11T06:12:56Z
draft: false
tags: ["xml"]
categories: ["class"]
---

## 70% 20%平时+80%期末  30%实验

## XSD

### XSD-\<schema>

\<schema> element is the root element of every XML Schema

``` xml
<?xml version = "1.0"?>

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
targetNamespace="http://www.runoob.com"
xmlns="http://www.runoob.com"
elementFormDefault="qualified"
>
...
...
</xs:schema>

```

### Simple element

`<xs:element name="xx" type="yyy"/>`

common types:

1. xs:string
2. xs:decimal
3. xs:integer
4. xs:boolean
5. xs:date
6. xs:time

simple element's defaults  
`<xs:element name="color" type="xs:string" default="red"/>`  
simple element's fixed values  
`<xs:element name="color" type="xs:string" fixed="red"/>`

### XSD Attribute

`xs:attribute name="xxx" type="yyy"/>`

common types:

1. xs:string
2. xs:decimal
3. xs:integer
4. xs:boolean
5. xs:date
6. xs:time

### XSD Restriction / Facets

The following example define a element name "age" with a restriction. The value of age cannot be lower than 0,higher than 120.

```xml
<xs:element name="age">
  <xs:simpleType>
    <xs:restriction base="xs:integer">
      <xs:minInclusive value="0"/>
      <xs:maxInclusive value="120"/>
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

Enumeration constraint

```xml
<xs:element name="car">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:enumeration value="Audi"/>
      <xs:enumeration value="Golf"/>
      <xs:enumeration value="BMW"/>
    </xs:restriction>
  </xs:simpleType>
</xs:element>

<xs:element name="car" type="carType"/>

<xs:simpleType name="carType">
  <xs:restriction base="xs:string">
    <xs:enumeration value="Audi"/>
    <xs:enumeration value="Golf"/>
    <xs:enumeration value="BMW"/>
  </xs:restriction>
</xs:simpleType>
```

Pattern constraint

```xml
<xs:element name="initials">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:pattern value="[A-Z][A-Z][A-Z]"/>
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

|restriction|description|
|---|---|
enumeration|define a list of acceptable values
fractionDigit|define max acceptable decimal places, >=0
length|define the quantity of acceptable char of list content
maxExclusive|<
maxInclusive|<=
maxLength|define the max length,>=0
minExclusive|>
minInclusive|>=
minLength|min length,>=0
pattern|define the exact sequence of acceptable characters
totalDigits|define the exact places of number,>0
whiteSpace|define the processing method of white space

### XSD complex elements

1. empty element
2. element containing other elements
3. element containing text
4. element containing other elements and text

#### EMPTY ELEMENT

```xml
<xs:element name="product" type="prodtype"/>

<xs:complexType name="prodtype">
  <xs:attribute name="prodid" type="xs:positiveInteger"/>
</xs:complexType>
```

#### ELEMENT CONTAINING OTHER ELEMENTS

```xml
<xs:element name="person" type="personType"/>

<xs:complexType name="personType">
  <xs:sequence>
    <xs:element name="firstName" type="xs:string"/>
    <xs:element name="lastName" type="xs:string"/>
  </xs:sequence>
</xs:complexType>
```

#### ELEMENT CONTAINING TEXT

```xml
<xs:element name="shoesize" type="shoetype"/>

<xs:complexType name="shoetype">
  <xs:simpleContent>
    <xs:extension base="xs:integer">
      <xs:attribute name="country" type="xs:string" />
    </xs:extension>
  </xs:simpleContent>
</xs:complexType>
```

#### ELEMENT CONTAINING OTHER ELEMENTS AND TEXT

```xml
<xs:element name="letter" type="letterType"/>

<xs:complexType name="letterType" mixed="true">
  <xs:sequence>
    <xs:element name="name" type="xs:string"/>
    <xs:element name="orderId" type="xs:positiveInteger"/>
    <xs:element name="shiDate" type="xs:date"/>
  </xs:sequence>
</xs:complexType>
```

### XSD Indicator

contains 7 types of indicator

Order Indicator

* All
* Choice
* Sequence

Occurrence Indicator

* maxOccurs
* minOccurs

Group Indicator

* Group name
* attributeGroup name
  
#### Order Indicator

1. All Indicator  
    every child element can occur in any order, and every child occur must occur once.

    ```xml
    <xs:element name="person">
    <xs:complexType>
        <xs:all>
        <xs:element name="firstName" type="xs:string"/>
        <xs:element name="lastName" type="xs:string"/>
        </xs:all>
    </xs:complexType>
    </xs:element>
    ```

2. Choice Indicator  
   one or the other

    ```xml
    <xs:element name="person">
    <xs:complexType>
        <xs:choice>
        <xs:element name="employee" type="employee"/>
        <xs:element name="member" type="member"/>
        </xs:choice>
    </xs:complexType>
    </xs:element>
    ```

3. Sequence Indicator  
    occur in order

    ```xml
    <xs:element name="person">
    <xs:complexType>
        <xs:sequence>
        <xs:element name="firstname" type="xs:string"/>
        <xs:element name="lastname" type="xs:string"/>
        </xs:sequence>
    </xs:complexType>
    </xs:element>
    ```

#### Occurrence Indicator

1. maxOccurs Indicator  

    ```xml
    <xs:element name="person">
    <xs:complexType>
        <xs:sequence>
        <xs:element name="full_name" type="xs:string"/>
        <xs:element name="child_name" type="xs:string" maxOccurs="10"/>
        </xs:sequence>
    </xs:complexType>
    </xs:element>
    ```

2. minOccurs Indicator

    ```xml
      <xs:complexType>
    <xs:sequence>
        <xs:element name="full_name" type="xs:string"/>
        <xs:element name="child_name" type="xs:string"
        maxOccurs="10" minOccurs="0"/>
        </xs:sequence>
    </xs:complexType>
    </xs:element>
    ```

### Group Indicator

1. Group Name Indicator

    ```xml
    <xs:group name="persongroup">
    <xs:sequence>
        <xs:element name="firstname" type="xs:string"/>
        <xs:element name="lastname" type="xs:string"/>
        <xs:element name="birthday" type="xs:date"/>
    </xs:sequence>
    </xs:group>

    <xs:element name="person" type="personinfo"/>

    <xs:complexType name="personinfo">
    <xs:sequence>
        <xs:group ref="persongroup"/>
        <xs:element name="country" type="xs:string"/>
    </xs:sequence>
    </xs:complexType>
    ```

2. Attribute Name Indicator

    ```xml
    <xs:attributeGroup name="personattrgroup">
    <xs:attribute name="firstname" type="xs:string"/>
    <xs:attribute name="lastname" type="xs:string"/>
    <xs:attribute name="birthday" type="xs:date"/>
    </xs:attributeGroup>

    <xs:element name="person">
    <xs:complexType>
        <xs:attributeGroup ref="personattrgroup"/>
    </xs:complexType>
    </xs:element>
    ```

### \<any>

