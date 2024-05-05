# Active Directory Domain Services

## Introduction

Windows **Active Directory** (AD) is Microsoft's solution to the problem of identity management. At the enterprise level solutions like AD are essential for scalable **Role Based Access Control (RBAC)** to company resources. Due to Microsofts ecosystem extending into many popular web applications, AD allowes users to integrate with a variety of services integral to an employees workflow. While Active Directory is still the most widely used **Identity Access Management** (IAM) it is confined to run on-premises, meaning that cloud native solutions like **Azure Active Directory** (AAD) will slowly become the new standard (likely making this article irrelevant in a few years).  

## Active Directory Services

AD is actually a suite of services that run on a Windows server, resist the temptation to force it to work on Linux. Typically when people refer to doing any kind of administration or cleanup in AD they are actually refering to **Active Directory Domain Services** (ADDS) which is responsible for the most used common functions like **Active Directory Users and Computers** (ADUC) and **Group Policy Management Console** (GPMC). ADDS essentially allows the domain to exist by installing all of the dependencies and foundation required for an AD environment.  

