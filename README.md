<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- Load properties -->
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:loanTypes.properties"/>
    </bean>

    <!-- SmartLoan Bean -->
    <bean id="smartLoan" class="com.spring.app.SmartLoan">

        <!-- customerId -->
        <constructor-arg value="12345"/>

        <!-- customerName -->
        <constructor-arg value="John"/>

        <!-- interestRatesMap -->
        <constructor-arg>
            <map>
                <entry key="personalLoan" value="${personalLoan}"/>
                <entry key="homeLoan" value="${homeLoan}"/>
                <entry key="carLoan" value="${carLoan}"/>
            </map>
        </constructor-arg>

    </bean>

</beans>
