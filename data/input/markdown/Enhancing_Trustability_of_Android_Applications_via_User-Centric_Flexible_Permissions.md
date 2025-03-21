

# Enhancing Trustability of Android Applications via User-Centric Flexible Permissions

# Gian Luca Scoccia, Ivano Malavolta, Member, IEEE, Marco Autili, Amleto Di Salle, and Paola Inverardi

Abstract—The Android OS market is experiencing a growing share globally. It is becoming the mobile platform of choice for an increasing number of users. People rely on Android mobile devices for surfing the web, purchasing products, or to be part of a social network. The large amount of personal information that is exchanged makes privacy an important concern. As a result, the trustability of mobile apps is a fundamental aspect to be considered, particularly with regard to meeting the expectations of end users. The rigidities of the Android permission model confine end users into a secondary role, offering the only option of choosing between either privacy or functionalities. In this paper, we aim at improving the trustability of Android apps by proposing a user-centric approach to the flexible management of Android permissions. The proposed approach empowers end users to selectively grant permission by specifying (i) the desired level of permissions granularity and (ii) the specific features of the app in which the chosen permission levels are granted. Four experiments have been designed, conducted, and reported for evaluating it. The experiments consider performance, usability, and acceptance from both the end user’s and developer’s perspective. Results confirm confidence on the approach.

Index Terms—Android permissions, static analysis, trustability

# 1 INTRODUCTION

THE Android market is experiencing an explosive growth in the last years. It is the smartphone OS market leader among all age segments in the US, UK, and other countries [1]. People rely on Android mobile devices for surfing the web, purchasing products, or to be part of a social network. Represented by the well known Google Play Store, the Android apps market now counts more than two millions apps, downloaded billions of times per year [2].

In this landscape, the Android permission model plays a key role, given the constant need of mobile apps to access sensitive and private information of end users [3], [4], [5], to the point that consumers have to choose between lower prices and more privacy protection [6], [7]. The Android permission model can influence the success of mobile apps as well. Indeed, a recent empirical study found that privacy-aware users with negative concerns on an app’s runtime permissions tend to give negative reviews to the whole app, thus confirming the importance of how permissions are managed for the success of Android apps [8].

G.L. Scoccia, M. Autili, A. Di Salle, and P. Inverardi are with the Department of Information Engineering, Computer Science and Mathematics (DISIM), University of L’Aquila, L’Aquila, AQ 67100, Italy. E-mail: {gianluca.scoccia, marco.autili, amleto.disalle, paola.inverardi}@univaq.it. I. Malavolta is with the Department of Computer Science, Vrije Universiteit Amsterdam, Amsterdam 1081, HV, The Netherlands. E-mail: i.malavolta@vu.nl.

Manuscript received 8 Jan. 2019; revised 15 July 2019; accepted 8 Sept. 2019. Date of publication 19 Sept. 2019; date of current version 15 Oct. 2021. (Corresponding author: Gian Luca Scoccia.) Recommended for acceptance by X. Zhang. Digital Object Identifier no. 10.1109/TSE.2019.2941936

The current Android permission model suffers of a number of rigidities related to, notably, the granularity level of the permissions, the timing at which permissions are granted, the fact that the permission model considers all users as equal [9], [10]. On the one hand, this lack of flexibility may have a negative impact on the success of a mobile app; on the other hand, it neglects the rights of the user to fully express her desires and exert her control on how and by whom her data are used. According to a survey conducted among adult Americans [11], 91 percent of participants believe that consumers have lost control over how personal information is collected and used by companies, and that most participants would like to do more to protect their personal information.

In this paper, we propose Android Flexible Permissions (AFP), a user-centric approach to flexible permissions management aimed at empowering end users to play an active role with respect to Android permissions. AFP embraces the European vision of next generation internet, more human-centric and concerned with privacy protection by giving control back to users [12]. End users are allowed to specify and customize fine-grained permission levels on private or sensitive resources, according to their own subjective privacy concerns, risk taking attitudes and sense of trust. AFP leverages a novel permission model through which fine-grained app permissions are specified on a per-feature1 basis.

Differently from the current Android permission model, AFP empowers end users to selectively grant finer-grained permissions by specifying (i) the desired permission levels2

1. Building on the definition provided in the Software Engineering Body of Knowledge [13], the term feature is used to denote a functionality offered by the app to the user.

2. The granularity at which a sensible resource/data can be accessed.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

2033

(e.g., access to the contacts list can be granted to all contacts that do not belong to specific circles of people like relatives or close friends), and (ii) the features of the app in which the specified permission levels are granted (e.g., access to the relatives circle in the contacts list can be granted only during the usage of the video call feature in a messaging app). AFP offers a dedicated external mobile app for managing flexible permissions.

From the developer’s point of view, AFP enables apps to support user-defined permission levels with very limited additional effort. Developers can work on their mobile apps as usual, without using any additional library or tool. In order to use AFP, a developer is provided with automatic support by the AFP Web application that (by means of a guided workflow) allows to (i) define the features offered by the mobile app; (ii) map each feature to the components that implement it, i.e., Android activities, services, broadcast receivers, or content providers. Given the feature-component(s) mappings, AFP leverages static control-flow analysis for automatically retrofitting the app so to make it able to handle fine-grained and feature-based permission levels.

We evaluated AFP by designing, conducting, and reporting four independent experiments aimed at empirically investigating on key aspects of AFP. Specifically, we assessed the performance of the AFP instrumenter via 1,277 real-world apps, the performance at runtime of 7 AFP-enabled real-world apps, the usability and acceptance of AFP for both end users and developers (involving 47 and 11 subjects, respectively).

The main contributions of this paper are:

1. an in-depth discussion of the characteristics and challenges of the Android permission model;
2. the definition of a new flexible permission model for Android apps;
3. the definition of an approach that empowers end users to specify and enact flexible permissions for Android apps;
4. a publicly available implementation of AFP using a combination of Java and Web technologies;
5. the results of the empirical evaluation of AFP, and a complete replication package for independently assessing and replicating the aforementioned experiments.

An initial version of this work appeared as a poster paper at the 2017 ACM/IEEE International Conference on Software Engineering. The new contributions of this journal version include: the precise definition of the AFP permissions data model, the app features specification and mapping mechanism, the algorithm for automatically instrumenting Android apps in order to make them AFP-enabled, and its implementation and technological aspects. Another completely new contribution of this paper is the empirical evaluation of AFP, for which we designed, conducted, and reported 4 independent experiments about its performance, usability, and acceptance from the perspective of both end users and developers. This paper also includes a thorough discussion of related work.

The remainder of the paper is structured as follows. Section 2 discusses in details the problem we want to solve and motivates our research. Section 3 presents the AFP approach, and Section 4 describes its implementation.

# 2 THE NEED OF A NEW PERSPECTIVE

In this section, we first analyze the evolution of the Android permission system by discussing the issues of both the older install-time permission system (Section 2.1) and the most recent usage-time permission system used by the latest Android releases (Section 2.2). Then, in Section 2.3, we discuss the need of a new perspective according to which users must be given the possibility of specifying their own rules on how sensitive resources should be accessed by apps.

# 2.1 Install-Time Permission System

Traditionally, up to version 5.1.1 (i.e., Android API level 22), Android makes use of an install-time permission system to regulate the access to sensible APIs of the platform. Developers have to declare upfront if their apps require access to security- and privacy-relevant parts of the platform. Very little control is in the hands of the end user, who can only decide to grant or reject all permissions to access such parts of the Android APIs before app installation, and can only rely on warning dialogs to assess possible risks (often using extremely broad wording). If the user only agrees with a subset of the permissions, she has to abort the app installation. This amounts to force the end user to either grant all permissions or not install the app. It has been observed that users routinely decide to not install an app because of the permissions it requires. Moreover, usability studies show that only a minority of users have a reasonable comprehension of warning dialogs. Dialogs have been found to be vague and devoid of context, as users have no way to know what app features the install-time permissions correspond to.

# 2.2 Usage-Time Permissions System

Starting with Android 6 (i.e., Android API level 23), access to privacy- and security-relevant parts of the platform can be enforced by a usage-time permission system. While the app is running, the system checks whether the app functionality that is going to be used for the first time has the required permissions or not. Users are prompted for confirmation when the functionality attempts to access a restricted part of the platform.

While runtime permissions provide more detail over the specific functionalities of the app affected by the permissions, hence helping users in making their decision, permissions are still granted on a whole-app basis, i.e., once granted the permission is valid for the entire app, further reducing control on how and when private data is accessed. Users can potentially revoke permissions already granted to an app but this requires exiting the app, accessing their device system settings and manually changing the permission settings of the app. The whole process is unwieldy, and users are unlikely to do so.

In addition, in the Android platform, permissions are grouped into permission groups. A permission group is a set of two or more permissions that reference the same.

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

Fig. 1. Overview of the AFP approach (underlined labels represent manual steps, whereas all the others are fully automatic).

resource. For instance, the permissions READ_CONTACTS and WRITE_CONTACTS both belong to the CONTACTS permission group. Whenever an app tries to access a protected resource, users are prompted for confirmation only if no other permission in the belonging permission group is already granted. This is done in order to minimize a phenomenon known as warning fatigue, which has been observed in the past in other usage-time permissions systems [18], [19], [20] – i.e., users may become insensitive to warning messages after being exposed to an excessive amount. While the concept of permission group can be effective in reducing warning fatigue, it also makes the permissions more coarse grained. Multiple permissions are in fact granted with a single confirmation, further reducing the user control on the resources that will be accessed with respect to the granted permissions.

# 3 THE AFP APPROACH

Android Flexible Permissions (AFP) grants permissions on a per-feature basis by (i) keeping track of user privacy preferences, and (ii) automatically enacting and enforcing them at runtime. AFP is composed of the following main components:

- AFP App, an app from which users can manage their own flexible permissions;
- AFP Library, a library for access control at runtime;
- AFP Server, a web app that allows developers to automatically retrofit an existing app in order to enable AFP in it. It also offers mechanisms for signing and verifying AFP-enabled apps.

With reference to Fig. 1, in the following we describe the workflow of AFP. Section 3.1 describes the steps to be followed by developers before publication, in order to make their apps compatible with AFP. Section 3.2 describes the...

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

steps to be followed by users upon the first app execution, in order to specify the desired permissions.

# 3.1 App Developer Perspective

The developer workflow is designed to minimize the effort needed to create apps compatible with the flexible permission system.

Developers create their mobile apps as usual, without using any additional library or tool. When an app X is ready to be published (right-hand side of Fig. 1), the developer can send the APK archive of X to the AFP Server so to enable AFP (1). As detailed in Section 3.5, the Android Components Extractor extracts all the Android components of X, i.e., its constituent activities, services, broadcast receivers, and content providers (2). Then, the developer uses a web-based editor for (i) defining the features of X in terms of their name and description (later used by end users), and (ii) mapping each one of them to (a subset of) the extracted Android components implementing it (3). Step 3 is the only additional effort we request to developers, and it is greatly facilitated by the web-based editor together with the automatic extraction of Android components. The output of this phase is the feature-component mapping model, specifying the mapping between app features and Android components.

The AFP Instrumenter statically analyzes and automatically retrofits app X to enable AFP on it (4). The instrumenter performs the following operations (that are totally transparent to the developer): (i) automatically includes our AFP Library in the app; (ii) instruments X so that all calls to sensitive Android APIs are proxified and redirected to the AFP Library; (iii) injects the code in the main activity of X for allowing the end user to switch to the AFP App when launching X for the first time; (iv) assigns a unique secret key to the app X, which will be used at runtime by the AFP App Checker (12); (v) creates a new record into the repository of registered apps; (vi) rebuilds and re-sign X as a new APK archive. Finally, the instrumented APK of X is made available to the developer (5), who can then proceed with the publication of the APK in the Google Play Store (6).

their reviews in the Google Play store [8]. Indeed, by explicitly asking end users to define the permission levels on a per-feature basis allows users to (i) read the description of each feature provided by the developer and (ii) better understand why certain permissions are being requested in the context of each specific feature of the app, rather than within the app as a whole. By using AFP, end users have a more transparent view of the features provided by their apps, and a better knowledge about the context in which (sensitive) permissions are requested by the app.

While the end user is setting her desired permissions, in background, the AFP App also interacts with the AFP Server (11). The AFP App and the AFP Server communicate via the HTTPS protocol. The server also uses an internally generated secret key (4) to check the app installation and verify the developer’s identity, hence certifying that nobody tampered with the AFP Library. Moreover, it verifies that the APK downloaded from the Google Play Store is exactly the one produced by our approach (12). When the results of the checks are ready (13), and the configuration phase finishes, the user will be redirected to the newly installed app, together with the configured flexible permissions configuration (14). The permissions configuration is then associated with the AFP-enabled app and the user can continue with app usage, in a completely transparent way, i.e., no further user interaction or dialogs are required.

The access to private or sensitive resources will be granted by the AFP Library according to the specified permissions configuration (15). The AFP Library proxifies each call of the app to sensitive Android APIs (e.g., call to the Android geolocation manager), hence wrapping the access to sensitive resources.

The AFP App allows to specify default levels for the permissions (e.g., geolocation is allowed only at the city-level, independently of the app requesting it), that will be used as a basis during the configuration of the flexible permissions for any newly installed AFP-compliant app. This characteristic permits to speed up the configuration of the permissions for each newly installed AFP-enabled app.

# 3.2 End-User Perspective

The user workflow is designed to minimize the end user effort to specify flexible permissions. Users can download and install (7) apps that adopt the AFP system directly from the Google Play Store since no modifications to the Android OS are required. Upon the first launch of the newly installed app (8) they are redirected to the AFP App (9), which in turn invites them to configure the flexible permissions. Should the user be unwilling to do so, she can immediately abort the process, and the app usage will continue with the standard permission system provided by the Android platform. The same happens if, during the configuration process, the AFP App is not found installed on the user’s device.

Once inside the AFP App, for each feature of the app the user can specify her own permission preferences (10). This aspect of AFP allows to address a well-known problem in the current Android permission model, i.e., the fact that users tend to not understand why they are being asked for certain permissions, often complaining about this aspect in their reviews in the Google Play store [8].

# 3.3 Flexible Permission Data Model

This section presents the AFP data model to which permission configurations conform (Fig. 2). It is based on the following core concepts:

Resource represents a sensitive part of the Android platform whose access can be controlled by the AFP library. In AFP, resources are both physical parts of the device, such as the device camera and microphone, and logical ones, such as the user’s contacts book.

Feature represents a user-level functionality of the app. Every Feature uses one or more Resources. In addition, every Feature is directly connected through the realizedBy relation with one or more AndroidComponent instances, each of them representing one or more source code files inside the app.

A PolicyItem regulates access to the Resources used by the Feature. It represents a single access restriction rule that can be imposed upon one or more Resources. For instance, a PolicyItem could specify that the...

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# 3.4 Features Specification

The AFP approach involves (i) the automatic extraction of Android components composing the mobile app (step 2 in Fig. 1) and (ii) the definition of a mapping between features and the Android components implementing them in the app (step 3).

For what concerns step 2, all relevant information is extracted from both the XML file of the Android manifest and the bytecode of the Java classes of the app. The output of this step is a fragment of configuration conforming to the flexible permission data model described in Section 3.3 containing only instances of the AndroidComponent class and its subclasses.

In order to specify the mappings (step 3), the developer uses a form (Fig. 3) where she can declare the main features of the app and, by means of check boxes, associate them to the automatically extracted Android components. The final result of this step is the complete configuration, which also includes the required instances of the Feature class.

# 3.5 App Instrumentation

The AFP approach involves also the automatic instrumentation of the app (step 4 in Fig. 1). Such step is carried on by the AFP Instrumenter that performs a set of operations that can be grouped into three main phases: (i) decomposing the input APK, (ii) rewriting the app bytecode, and (iii) repackaging the rewritten app.

The goal of the first phase is to extract, from the compiled binary files of the app under analysis, the app Java bytecode. Although the bytecode is a low-level representation, it is suitable to perform analysis and instrumentation. In turn, the third phase performs the reverse operation, transforming the instrumented bytecode back into a compiled binary file. The logic behind the first and the third phases is straightforward, and they are both carried out using freely available tools arranged in a pipe-and-filter pattern. The adopted tools are described in Section 4.

# Feature

|Name|Description|
|---|---|
|Blog|With this feature you can create and edit posts, and pages; and upload media|

# Android Components

|Hame|Type|
|---|---|
|org.wordpress.android.ui.WPLaunchActivity|activity|
|org.wordpress.android.ui.main.WPMainActivity|activity|
|org.wordpress.android.ui.accounts.NewBlogActivity|activity|
|org.wordpress.android.ui.posts.services.PostUploadService|service|
|org.wordpress.android.ui.posts.services.PostMediaService|service|

Showing 1 to 5 of 5 entries

Previous Hext

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

# Algorithm 1. Rewriting Algorithm of AFP Instrumenter

Input: App; an Android app

C; set of Android components 2 App

F; set of features 2 App

M; mapping of elements of F to C

Output: App0; AFP-compliant version of App

1: begin
2:    foreach c 2 C do
3:      scan C to extract SC, set of sensitive calls 2 C
4:      if SC 6 ¼ ; and c = M then2
/* found sensitive call not mapped to any
feature, raise error and terminate */
5:         break
6:      end
7:      if c is App main activity then
8:         add intent trigger towards AFP app to
allow for permissions configuration
9:      end
10:      foreach sc 2 SC do
11:         identify r, resource affected by sc
12:         foreach f 2 F do
13:            if c is mapped to f then
14:               add r to f , set of resources used by fr
15:            end
16:         end
17:         replace sc with sc0, call to AFP library wrapper
18:      end
19:    end
20: end

# 3.6 Permissions Enactment and Enforcement

As described in Section 3 (see Fig. 1), a user can download and install AFP-enabled apps from the Google Play Store, as she would normally do for all apps. Upon starting a newly installed app for the first time, she is redirected to the AFP App, which enables permission enactment by allowing her to configure the flexible permissions associated to it. Two screenshots of the app are presented in Fig. 5. During the configuration, she is presented the list of features offered by the AFP-enabled app. For each feature, the accessed sensitive resources are listed and, for each one of them, she can set her preferences, hence regulating the access to the resources for that single feature. As an example, consider a user interested in the Facebook app. After downloading and installing it on her device, on the first run she is presented with the list of app features, i.e., Wall, Messaging, Events, etc. Assuming that she does not want her friends to know her exact location every time she posts on her Wall, she can restrict the precision of the Location resource for the Wall feature, while leaving it unchanged for Events in order to still discover ongoing events nearby.

During the second phase, rewriting of the app bytecode is performed by an ad-hoc analyzer, following Algorithm 1. Given as input an Android app and its feature-component mappings, the algorithm returns an AFP-compliant version of the app. In order to do so, the algorithm iterates over all Android components that constitute the app (line 2 in Algorithm 1) and extracts from each of them the set of sensitive API calls performed inside its logic (line 3). Each call is analyzed for the purpose of identifying the affected resource (lines 10-16), and creating and binding instances of the Resource class in the AFP data model. Each sensitive call is then replaced with a corresponding call to the AFP library (line 17), which contains a proxy class for each of the Android APIs that enable access to restricted parts of the platform (further details in Section 3.6). While performing the rewriting, the algorithm also checks that all classes containing calls to restricted parts of the Android API belong to a feature (lines 4-6). This ensures that the developer does not leave some Android components out of the mapping. Additional code (via Android Intents) is added in the main activity (i.e., the app entry point) to set up communication between the app under rewriting and the AFP App. It enables the configuration of the flexible permissions on the startup of the app (lines 7-9).

The illustrative example, given in Fig. 4, shows bytecode rewriting of an API call for reading the last registered user’s location. In the original version (Listing A), a Location-Manager object is loaded from a local variable and pushed onto the stack (instruction 1). Then, the constant string “network”, used as a parameter in the upcoming method invocation, is also pushed onto the stack (instruction 2). Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13,2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# 4 IMPLEMENTATION AND USED TECHNOLOGIES

AFP makes use of a number of different techniques and technologies. Static analysis techniques are used to verify APK packages uploaded by developers on the AFP Server. Specifically, static analysis is utilized to verify that developers accessed sensitive resources only through the methods provided by the AFP Library (otherwise we cannot guarantee that the preferences set by users in permissions configurations will be fulfilled). In order to do so, AFP utilizes an intra-procedure analysis to detect Android API invocations within each method of the app’s code. Our prototype implementation adopts the list of sensitive Android APIs provided by Rasthofer et al., who also provide a methodology to obtain an up-to-date list. For the implementation of this analysis, we rely on the static analysis framework Soot, coupled with Dexpler to disassemble APK packages and transform Android’s Dalvik bytecode into a format suitable for analysis.

The integrity of the AFP Library implementation is also checked by the AFP Server through a checksum-based integrity verification mechanism. This permits to avoid the possibility that malicious developers could tamper with the library, thus circumventing the need of obtaining authorization against the permissions configuration.

Communication between the AFP App and any AFP-compliant app is enabled by Android’s Intents. An Intent is a special kind of object used to enable inter-app communication. In AFP, explicit Intents are leveraged to both redirect the end user to the AFP App and return the configured fine-grained permissions configuration once it has been personalized.

Changes to the permission system introduce the risk of app instability, as apps may not expect to have their permission requests denied. When denying permissions leads to crashes, users are likely to become more permissive to improve app stability, thus counteracting the whole reasoning behind feature- and level-based permissions. With this concern in mind, in our implementation, we make use of “mocking”: in the event of a denied permission our system supplies apps with well-formed but non-sensitive data. For example, if the end user allows only city-level geolocation, when the app calls the Android location manager, the AFP Library intercepts that call and returns the geographical center of the city where the user is, instead of her precise location. This enables apps to continue functioning usefully unless access to the protected resource is critical for its correct behavior.

To perform the automatic extraction of the Android components composing the mobile app (step 2 in Fig. 1) we first decode the input APK using apktool. The Android manifest file is then analyzed via a simple XML parser we developed in Java. The analyzer of the Java bytecode is implemented by using the Apache Commons Byte Code Engineering Library.

The tool that allows developers to specify feature-components mappings (step 3) has been implemented as a web-based tool, built upon the Flask web framework.

3. This aspect is inspired by the Mockdroid approach by Beresford et al.

4. http://ibotpeaches.github.io/Apktool/

5. http://commons.apache.org/proper/commons-bcel/

6. http://flask.pocoo.org

# Fig. 4. Comparison between an original byte code file (Listing A) and the rewritten version produced by the AFPInstrumenter (Listing B).

# Fig. 5. Two screenshots of the AFPApp: List of features presented to the end user and possible levels for Location.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

Fig. 6. Android software stack with added AFP components.

The AFP Instrumenter is implemented by using several tools. The tool apktool is used for decomposing the APK and producing a Classes.dex file containing the app bytecode. Then, the dex2jar tool is used to obtain a conventional jar file that, subsequently unpacked via the zip shell command, permits to obtain the .class files constituting the app. Rewriting of the .class files is done by our Java implementation of Algorithm 1, leveraging the Apache Commons Byte Code Engineering Library. Instrumented .class files are then repackaged back to a .dex archive via the Android SDK dx tool, and the APK archive is reassembled using again apktool. At the end, the resulting package is signed using jarsigner. The whole end-to-end process is tied together by a Python script.

# 5 EVALUATION

In this section, we report the four independent experiments we performed to evaluate the AFP approach. For the purposes of the experiment 2, 3 and 4, we focused on the three Android APIs that are among the ones considered the most sensible by end users while at the same time widely used by apps on the Google Play market: Camera, LocationManager, and MediaRecorder. To allow easy replication and verification of the experiments, we provide a complete replication package including: the source code of all the components of the AFP approach, the source code of the measuring tools we implemented for carrying on the experiments, the raw data we obtained from the experiments, and all the scripts for analysing the experiments’ results.

# 5.1 Experiment 1: Performance of the AFP Instrumenter

Design. The goal of this experiment is to assess the performance of the AFP Instrumenter, the module of the AFP server that performs the app static analysis and instrumentation. We chose the AFP Instrumenter as the object of our experiment since it is the most complex component in our AFP Server, and its malfunctioning or low performance in terms of execution time may negatively impact the adoption of the whole approach by developers, who will not be willing to spend (relatively) long time for the result of the app instrumentation phase.

Results. Fig. 7 shows the execution times of each step of the AFP Instrumenter pipeline. Each step takes an average of less than 10 seconds, with the only exception of the dx tool (18.78 seconds in average), mainly because of its heavy I/O operations and performed optimizations. When considering the total execution time of the whole pipeline, we can observe that our AFP Instrumenter takes an average of 41.19 seconds to complete, with a minimum of 4.03 seconds and a maximum of 143.07 seconds.

Discussion. We consider the results as satisfactory. On average, developers have to wait less than a minute for obtaining the AFP-enabled app from the AFP Server, and less than 3 minutes in the worst case of our experiment. Since the AFP Instrumenter is executed only once for each app, the waiting time for developers due to app instrumentation is reasonable. Hence:

The performance of the AFP Instrumenter is satisfactory, requiring on average less than a minute.

7. http://github.com/pxb1988/dex2jar

8. http://wing-linux.sourceforge.net/guide/developing/tools

9. http://docs.oracle.com/javase/tutorial/deployment/jar/signing.html

10. https://github.com/gianlucascoccia/d1f8de71bcdc39d1f4827c04a952a0c29/dx/src/com/android/dx/androidflexiblepermissions

11. https://android.googlesource.com/platform/dalvik/+/a9ac3a9

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# 5.2 Experiment 2: Performance of AFP-Enabled Apps

Design. The goal of this experiment is to assess the performance of the AFP-enabled apps at runtime (i.e., app X in Fig. 1). The main rationale behind this experiment is to assess how the application of our AFP approach may actually impact the performance of instrumented apps, thus potentially impacting the overall user experience.

For this experiment, we selected 7 mobile apps from a publicly available dataset composed of 2,443 open-source Android apps that are freely distributed in the Google Play store and whose source code is hosted on GitHub [32]; these two conditions permitted us to have a dataset with apps designed and developed as real projects with real users, and to easily check that instrumented apps behave as the original ones (we did this by a combination of source code inspection and the addition of logging instructions in key parts of the app). Among the 2,443 apps, we randomly selected the 7 apps (see Table 1) among those requesting at least one of the permissions considered for our experiments (i.e., geolocation, camera and microphone access).

**Selected Apps for Study 2**
|ID|Name|Version|Type|Scenarios|
|---|---|---|---|---|
|a1|WordPress|5.6.1|Blog manager|2|
|a2|Ottawa Bus Follower|2.0.11|Bus-related utility|2|
|a3|Streetlight Seattle Reporter|1.2.0|Citizen participation|2|
|a4|Local Weather|1.0.0.7|Weather|1|
|a5|Run Helper|1.3|Fitness tracker|1|
|a6|Selfie HD|1.1|Photo camera|1|
|a7|Flickr Uploader|2.3.2|Photo upload|1|

We executed the experiment by performing the following steps for each app: (i) we defined one or two (depending on the complexity of the app, see Table 1) common usage scenarios that start from the main activity and end with the complete stop of the app; (ii) we executed each usage scenario, while measuring the CPU load and memory consumption of the process of the app; (iii) we created a feature-component mappings using the AFP Web application; (iv) we instrumented the app via the AFP Instrumenter by using the mapping defined in the previous step; (v) we executed and measured again each usage scenario on the instrumented version of the app.

Threats to Validity. A possible threat to validity of our experiment is represented by the selection of only free apps as subjects. Although this choice was driven by budgetary constraints, free apps are representative as they represent 75 percent of all Google Play Store apps and are downloaded more often [31] than paid apps.

A second threat is represented by the fact that, in this experiment we consider only one kind of hardware machine. This choice is mainly guided by budget constraints related to both time and available resources. Notwithstanding, the used hardware is consumer-grade, hence we believe that collected measures are representative of performances that can be obtained on ordinary hardware. Moreover, it is important to note that the AFP Instrumenter is deployed in the AFP Server, whose hardware and software performance can be far higher than the machine we used for this experiment and can be easily scaled up if deployed in a virtualized/containerized environment.

Results. Collected measurements are presented in Fig. 8. For each app, we performed a comparison of both the CPU load and memory consumption for its two versions (i.e., original or instrumented) by using the Mann-Whitney test [29] with a = 0.05, one-tailed. In all cases, we obtained a p-value much larger than a, thus allowing us to confirm that there is low difference in the medians of CPU and memory consumption, with and without app instrumentation.

Discussion. From Fig. 8 it is evident that both CPU load and memory consumption of the original and instrumented versions of each app are comparable, as confirmed by statistical tests. The results of this experiment give a positive indication about the performance of AFP-based apps.

It is important to note that the focus is not on the formal systematic assessment of the precision of the app instrumentation (i.e., we do not have a formal proof that instrumented apps do not crash in some corner cases); nevertheless, we performed a manual assessment of stability of the 7 apps in experiment 2 by performing a set of evaluation runs, observing that the instrumented app conformed to the expected behavior. To recap: Performance of AFP-enabled apps are comparable to those of regular apps.

Threats to Validity. One common risk to validity of the experiment is the threat that adopted feature-component mappings and execution scenarios may not be representative of real app usage. To mitigate this threat, as a preliminary step, selected apps were examined by (i) analysing apps description in the Google Play store, (ii) manually inspecting their source code, and (iii) performing a set of preliminary runs while observing app behavior. Two different researchers were involved in the definition of both artifacts: first they were proposed by one and then verified to be representative by a second one.

A second threat is represented by the limited amount of apps involved in the experiment (7), a very small minority.

12. https://android.googlesource.com/platform/sdk/+/6db5720/monkeyrunner/src/com/android/monkeyrunner/recorder/MonkeyRecorder.java

13. http://developer.android.com/studio/test/monkeyrunner

14. http://developer.android.com/studio/command-line/adb.html

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

2041

# 5.3 Experiment 3: Usability and Acceptance of AFP by Developers

Design. The goal of this experiment is to evaluate the usability and acceptance of AFP by developers. For this purpose, we conducted an on-line study, involving Android developers, in which we asked them to build the feature-component mappings for the apps they developed. We focused on this aspect as it is the main effort required to developers to make their app AFP-compliant (all the other steps are automated). This experiment is composed of three main phases:

1. Recruiting: we posted an announcement on pertinent on-line discussion groups (i.e., Android developer forums, mailing lists) to enlist developers willing to take part in the study. Each developer was asked to provide (at least) one link to an app published on the Google Play store, on which (s)he worked (either alone or in a team).
2. Mapping: the participants who provided a working link in the previous step were then invited to access a web-based app containing: (i) the definitions of feature- and level-based permissions, and (ii) the AFP web editor for the feature-component mappings. The participants were instructed to create the feature-component mappings for one of their previously-linked apps, and we collected the mappings defined by developers. We kept track of the time required by each participant for creating the feature-component mappings.
3. Evaluation: after completing the mapping, participants were asked to complete an on-line evaluation questionnaire about AFP. In order to encourage developers in participating to the experiment, we informed them that, during the mapping step, they did not have to manually input the activities of their Android app, as we had preemptively loaded them in the web editor after downloading and analyzing the manifest file of each app collected in the recruiting step.

Results. A total of 11 developers completed both the mapping definition and the evaluation questionnaire, providing us with twelve mappings in total (one developer performed the mapping construction for 2 apps). Developers participating in the experiment are also quite heterogeneous, both in terms of experience, number of developed apps, and size of organization. Specifically, participants have an average of 3.45 years of Android development experience (standard deviation = 2.66) and their majority (5) developed between 2 and 5 Android apps during their career, followed by 2 developers who developed more than ten. For what concerns organizations, the majority of developers work in small organizations (i.e., with 2 to 10 members), but we have also participants working in organizations with a number of members between 10 and 50. Finally, six developers declared to be Satisfied with current Android permissions, three declared to be Unsatisfied and two are Unsure.

# TABLE 2

Structure of the Evaluation Questionnaire Used for Developers

|Evaluation Goal|Question ID|Question Text|
|---|---|---|
|Acceptance|q1|Is the definition of level-based and feature-based permissions clear to you?|
|Acceptance|q2|Do you rate the definition of level-based and feature-based Android permissions as useful?|
|Acceptance|q3|Are you willing to use feature- and level-based permissions in your apps?|
|Usability|s1-s10|As defined in the System Usability Scale [33].|

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# TABLE 3

# Apps Developed by Participants of Experiment 3

|Developer ID|App ID|Package Name|Version|Type|Relevant resources|# Features|Mapping time (s)|
|---|---|---|---|---|---|---|---|
|D1|A1|com.rubikssolutions.daily5sec|3.2|Video sharing|Camera, Microphone|2|21|
|D2|A2|com.ambiensvr.mobile|0.9|Augmented reality|Camera|4|924|
|D3|A3|fr.inria.es.electrosmart|1.6|Electromagnetic waves meter|Location|8|337|
|D3|A4|ums.lovely.university|9.8.4|University management system|Camera, Location|4|1809|
|D4|A5|com.digitech.foodel|1.9|Food delivery|Camera, Location|22|1758|
|D5|A6|com.mobile.wabi_tech.gadfly|2.2.9|Citizen participation|Location|3|49|
|D6|A7|com.Jitendra93266.jitu.knowmovies|1.0|Movies database|-|3|54|
|D7|A8|de.jw.mymensa|0.9.0|Menu viewer|-|2|47|
|D8|A9|com.peaklens.ar|1.0.11|Augmented reality|Camera, Location|7|455|
|D9|A10|com.yopapp.yop|1.9.4|Online marketplace|Camera, Location|4|192|
|D10|A11|com.myoxygen.press.weaf|3.0|Aerospace news|Location|4|61|
|D11|A12|ro.notnull.IdenticalFilesFinder|3.4.0|System management|-|4|80|

mappings, developers took an average of 482.25 seconds, i.e., 8.03 minutes (median = 136s, min = 21s, max = 1809s, SD = 660s). Even in the worst case, the time required by the participants to create the feature-activities mapping is close to half an hour. We consider such amount of time acceptable, considering that the definition of such mapping is conducted only once for an app.

Fig. 9 summarizes the distribution of answers for questions q1, q2 and q3. When developers were asked about whether they understood the concepts behind feature- and level-based permissions (q1), only one developer answered No, three answered Absolutely Yes and the remainder Yes. useful (q2). Summarizing: On a similar note, when asked whether they consider feature- and level-based permissions as useful (q2), answers were two Absolutely Yes, five Yes, one Don’t know, two No and one Absolutely No. Concerning whether they would be willing to use the AFP permissions in their apps (q3) answers were two Absolutely Yes, four Yes, two Don’t know, two No and one Absolutely No. For all three questions median value of answers is Yes.

Results of the usability part of the evaluation questionnaire are presented in Fig. 10, where each column of the heatmap represents the distribution of answers for each of the ten statements that comprise SUS. The procedure described in the SUS guidelines [33] was applied to normalize answers to each statement. For most statements, answers are clustered towards the middle of the scale for.

Fig. 9. Results about the acceptance of AFP by developers.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

the majority of statements, revealing an acceptable usability. DOP Further confirmation is found in the obtained mean SUS score (49.77) that, according to prior research [34], is to be evaluated as Ok. Exceptions are statements s3 and s4. Most participants are in agreement with the former, that reads “I thought the system was easy to use”, confirming that the process was not overly difficult. At the same time developers felt in agreement with the latter: “I think that I would need the support of a technical person to be able to use this system”. Such results, and the fact that developers are not always familiar with permission-related technical aspects [36], encourages us to investigate in the future on techniques to assist in (or even to automate) the definition of feature-component mappings. Summarizing:

Developers positively evaluated the usability of the AFP approach. Improvements can be made on making the definition of the feature-component mappings more straightforward from a technical perspective.

# Threats to Validity

Possible threats to the validity and points of improvement for experiment 3 are as follows. The number of participants in the experiment (11) represents a very small minority of mobile app developers in the real world, hence results might not generalize. This threat is mitigated by the fact that developers who participated in the experiment have varied years of experience, nationalities and work in organizations of different size. A second threat is represented by the limited number of apps for which developers created the mappings. Thus, the results of our study might potentially not generalize to other apps. This threat is mitigated by the fact that submitted apps have very different purposes, provide different features, and have been developed by different developers. Finally, as the study has been conducted on-line, we had no way to ascertain that participants fully understood the task they were asked to complete. We mitigated this potential threat by directly asking in the final questionnaire whether developers had additional comments or doubts to clarify with respect to AFP.

In order to keep the experiment as representative as possible, we decided to reuse three of the apps for which real developers provided a mapping in Experiment 3. The app selection was performed with the goal of having at least one app making use of each of the device resources currently adopted by the current implementation of AFP (i.e., Camera, Microphone, and Location). Unfortunately, we could not successfully instrument app A1, the only one in our dataset that uses the microphone, because it relies on Java reflection (a language construct traditionally hard to deal with by approaches relying on static analysis [37]). Hence, we discarded it and selected an alternative app, thus ending with apps A3, A10, A11 in our final app selection.

# 5.4 Experiment 4: Usability and Acceptance of AFP by End Users

Design. The goal of this study is to evaluate usability and acceptance of AFP from the end-user perspective. We conducted an in-person study involving 47 participants. This experiment is composed of three main phases:

1. Pre-study: each participant was given a short description about the goal of the experiment, together with the definition of feature- and level-based permissions. Demographic information was also collected.
2. First trial: each participant was asked to try out three Android apps, employing either AFP or the current Android permission system. An evaluation questionnaire was given at the end of the trial.
3. Second trial: the participant was asked to repeat the trial, this time employing the permission system that was not used during the previous phase. Again, an evaluation questionnaire was given at the end of the trial.

The first part of the questionnaire (Trustability) consists of two questions (Q1, Q2). It was filled by participants at the end of both the second and the third phase of the experiment. The second part of the questionnaire (Acceptance) evaluates the acceptance of AFP and the last one assesses usability of AFP.

A breakdown of the structure of the evaluation questionnaires provided to users is shown in Table 4.

# 2044

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# TABLE 4

# Structure of the Evaluation Questionnaire for End Users

|Evaluation goal|Evaluated for|Question ID|Question text|
|---|---|---|---|
|Trustability|AFP vs Android|Q1|How do you judge the way the app asked for permissions?|
| | |Q2|How likely are you to use the app on your device, considering the permissions you were asked for?|
| | |Q3|Is the definition of feature-based permissions clear to you?|
| | |Q4|Is the definition of level-based permissions clear to you?|
|Acceptance|AFP|Q5|Do you rate feature-based permissions as useful?|
| | |Q6|Do you rate level-based permissions as useful?|
|Usability|AFP|S-S101|As defined in the System Usability Scale [33].|

# TABLE 5

# Breakdown of End Users Privacy Preferences

|Feature|Resource|Access level (%)|
|---|---|---|
|F1|Location|Full access: 19 (40%)|
| | |Region only: 6 (13%)|
| | |City only: 17 (36%)|
| | |Deny: 5 (11%)|
|F2|Camera|Allow: 38 (81%)|
| | |Deny: 9 (19%)|
|F3|Location|Full access: 4 (9%)|
| | |Region only: 17 (36%)|
|F4|Location|City only: 23 (49%)|
| | |Deny: 3 (6%)|
| | |Full access: 5 (11%)|
| | |Region only: 20 (43%)|
| | |City only: 18 (38%)|
| | |Deny: 4 (9%)|
| | |Full access: 9 (19%)|
| | |Region only: 15 (32%)|
|A11|Location|City only: 10 (21%)|
| | |Deny: 13 (28%)|

contains four questions (Q3-Q6) and it was filled by participants after completing the third phase of the experiment. All answers to questions Q1-Q6 range on a five-point Likert scale.

Also the third part of the questionnaire (i.e., Usability) was filled by participants after completing the third phase of the experiment. In this part, we relied again on the System Usability Scale (S1-S10). At the end of the questionnaire, open comments were collected from the participants.

Results. Professions and backgrounds of participants are varied and include students, shop assistants, mechanical engineers, lawyers, etc. Participants were mostly male (68 percent) and the mode of their age is Between 21 and 30 years old. Although not perfectly balanced, we consider the group of participants to be a good sample of the general smartphone users population, itself skewed towards younger ages and males [38]. The mean self-assessed knowledge of Android is 3.43 on a 1-5 scale (1.17 standard deviation). Roughly half of the participants (24/47) tried out the apps with the Android permission system as first, and with AFP as second. The opposite order was adopted for the others.

Fig. 12 summarizes the distribution of answers for questions Q1 and Q2 for both AFP and the Android permission system (recall that participants were asked these questions twice). For both questions, users provided more favourable answers for AFP, with a median value of Trustable for Q1 and Likely for Q2, as opposed to Android which achieved a median of Neutral for both questions. Obviously, differences in answers are statistically significant, which we confirmed by performing the two-tailed Mann-Whitney U-test [39].

# Fig. 12. Perceived trustability of Android and AFP permission systems w.r.t. the way the app asked permissions (Q1) and how likely the participant is to use the app (Q2).

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

# 2045

Very trustable Very likely

Trustable Likely

Neutral 1 Neutral

Not trustable Unlikely -

Absolutely not trustable

Android AFP Android AFP

Android knowledge Android knowledge

Fig. 13. Perceived trustability (Q1) and likeliness to use the app w.r.t. the way the app asked permissions (Q2) by self-declared Android knowledge.

We obtained a p-value of 7:623e08 for Q1 and 4:802e05 for Q2, thus rejecting the null hypothesis that the distributions of the answers about Android and AFP are equal. Fig. 13 provides a breakdown of answers to Q1 and Q2 by self-declared Android knowledge of participants. For both questions, AFP achieved more favourable answers even for participants with a lower level of knowledge. Answers for the Acceptance part of the questionnaire (i.e., Q3, Q4, Q5 and Q6) are summarized in Fig. 14. The answers are skewed towards the positive part of the scale and the median value is Absolutely yes for most of them, with the only exception of Q5 for which the median value is Yes. Fig. 15 provides an overview of answers to the same questions by self-declared Android knowledge of participants. For all levels of knowledge, the majority of answers fall in the upper end of the scale.

Results of the usability evaluation are shown in Fig. 17. Each column of the heatmap presents the frequency distribution of answers for one of the ten SUS statements. The procedure described in the SUS guidelines [33] was applied to normalize answers to each statement. For all ten statements, participants provided mostly positive answers with the total amount of negative ones being less than 15 percent.

Still concerning users choices, it is worth noting that, when possible, users favoured more restricted access levels and seldom provided full location access. An exception is represented by feature F1, for which full access is the consensus choice. During the execution of the experiment, we observed that users believed that full access was strictly necessary for performing this feature, thus explaining such difference. Consequently, we can state that: When possible, most users choose levels that disclose a restricted amount of data.

From the answers to questions Q1 and Q2 (see Fig. 12), we can point out that users generally felt more secure when employing AFP and praised the added control over shared personal data provided by it. Fig. 13 provides an indication that this preference towards AFP is constant across varying levels of Android knowledge. Such considerations are also supported by some of the comments collected during the evaluation questionnaire: one participant stated that “(Between the two systems) I prefer AFP as I have more control on permissions and I feel more secure”, while another participant pointed out that “as a user, with AFP I am more aware on.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# Acceptance of AFP Q3-Q6 by self-declared Android knowledge

|Absolutely yes|Yes|Don't know|No|Absolutely no|
|---|---|---|---|---|
|2 3 4 5 1 2 3 4 5 2 3 4 5 1 2 3 4 5|2 3 4 5 1 2 3 4 5 2 3 4 5 1 2 3 4 5|2 3 4 5 1 2 3 4 5 2 3 4 5 1 2 3 4 5|2 3 4 5 1 2 3 4 5 2 3 4 5 1 2 3 4 5|2 3 4 5 1 2 3 4 5 2 3 4 5 1 2 3 4 5|

# Usability of AFP (S1-S10) by self-declared Android knowledge

|Strongly agree|Agree|Neutral|Disagree|Strongly disagree|
|---|---|---|---|---|
|Android knowledge|Android knowledge|Android knowledge|Android knowledge|Android knowledge|

For odd numbered statements higher is better. For even numbered statements lower is better.

Users feel more secure and are more willing to use apps on their smartphone when using feature- and level-based permissions. This leads us to the following reflection point: misleading and enforce a binary choice. I appreciated AFP’s level-based permissions. In conclusion:

Feature- and level-based permissions are both well understood and deemed useful by end users, although the usefulness of the latter is more immediate to them.

Fig. 14 outlines the distribution of answers to the acceptance part of the evaluation questionnaire (questions Q3 through Q6). The definitions of feature- and level-based permissions were both well understood and deemed useful by users, although the former achieved worse answers regarding its perceived usefulness (Q5). We conjecture this difference is due to the fact that the perceived benefit of level-based permissions is more immediate to users. Fig. 15 suggests that, also in the case of acceptance, the distribution of the answers is unaffected by the users’ knowledge of Android. Further confirmation of the perceived usefulness can be found in the open comments: one participant stated “I appreciated the greater choice of options provided by AFP”; a second one noted that “(Android) permission pop-ups are misleading.”

Regarding the results of the usability evaluation, according to prior research [34], the achieved mean SUS score (78.19) is to be interpreted as a Good level of usability (see Fig. 17). Focusing on the distribution of answers for each statement, positive answers are the majority for all statements, even for users with a limited knowledge of the Android platform (see Fig. 16). Nonetheless some points of improvement can be identified. Statements S1, S2, S3 and S5 achieved a comparatively lower amount of maximum score answers, revealing that some users did experience some difficulties while using the system. Investigating the comments left by participants, we noticed that some users would prefer to grant feature- and level-based permissions at runtime. In particular, one user noted “I would like to grant permissions.”

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# LIMITATIONS AND FUTURE WORK

The proposed approach suffers a number of limitations that we plan to address in the future. At the time of writing, AFP only permits the specification of PolicyItems that reference a single resource. Our plan is to enhance the permission model by introducing consistency-checking of policies and reasoning on multiple resources (e.g., do not allow access to contacts book if the app also requests Internet access).

As previously discussed in Section 3.6, our current implementation of AFP is fully contained in the application layer of the Android stack. Our prototype implementation was developed with the main goal of assessing the perception of users on the proposed feature- and level-based permissions. We are aware that a full market-scale adoption requires a more tightly-coupled integration with the lower levels of the Android stack. In this direction, the integration of AFP with existing works on kernel-space security would be beneficial.

Furthermore, API wrappers are currently developed on a per-API basis. In the future, we plan on investigating possible categorizations of Android APIs, in order to group together similar ones, hence reducing the effort required to develop new wrappers. As these groupings must be well understood by both end users and developers, we plan on conducting new experiments to assess their effectiveness.

Threats to Validity. There are several threats and points of improvement for Experiment 4. Although users were instructed to act as they would do with their own smartphone, they were still performing the trials in a controlled environment, potentially different from the normal. This means that participant activities and answers may differ from what can be observed in the real world. As future work, we will mitigate this potential threat to validity by performing the experiment via an app that users can install on their own smartphone in order to monitor the AFP-enabled apps during their usage.

Despite our efforts to have a balanced and unbiased set of participants, we ended up with a group of relatively young people (age between 21 and 30 years old) and with a majority of male participants (32 males as opposed to 15 females). Moreover, we are aware that the sample size of this experiment (47 participants) is limited with respect to all Android smartphone users today. We mitigated this potential threat by contacting participants with different backgrounds and professions, different experience about the Android platform, and by letting them interact with the real apps instead of Android emulators or simulated services to directly publish mobile apps to the Google Play Store.

Another possible threat is represented by the limited amount of apps used to perform trials in the study. Hence, results of our study might potentially not generalize to other apps. We mitigated this threat by selecting three apps with very different purposes and features, thus collecting data on users behavior in varied scenarios. Additionally, participants were not familiar with selected apps in advance. Hence, their behavior during the trial might properly resemble the one of a user that runs a freshly installed app for the first time.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# 7 RELATED WORK

Android permissions and their usage have attracted considerable research interest over the years. In [3], Felt et al. report on the usability study they performed to evaluate whether Android users pay attention and understand permissions upon installing apps. The outcome of the study is that Android users make security and privacy decisions with a minimal comprehension of what the implications of their decisions will be.

The problem addressed in our paper falls into the class of problems identified in [3], and from there it takes the move. The authors of [7] investigated whether users are willing to pay premiums for privacy: results suggest that many smartphone users are concerned with their privacy and are willing to pay premiums for apps that are less likely to request access to personal information. Similar conclusions are presented by Kelly et al. in an independent study [15] that, in the form of a series of semi-structured interviews, was aimed at understanding user’s attention to and comprehension of permissions screens displayed during app’s installation. Their findings highlight that, while permissions displays are generally viewed and read, mostly often they are not understood by Android users.

Recent work has focused on studying the new Android runtime permission system. In [8], we identified points of possible improvement in the new system analyzing mobile apps user reviews. In [46], Peruma et al. conducted an in-person study to investigate user perception of the previous, current and a new proposed system. Among their findings there is the indication that current system does not make users feel more secure than the old one.

Andriotis et al. studied users’ adaptation to the new runtime permission model [47], collecting and analyzing anonymous data from 50 participants. Although their analysis indicates that users adapted positively to the new model, participants of their study hardly ever utilized the newer capabilities that allow a more efficient control over resources used by applications [17]. In [48], the same authors suggest that users should be informed about the resources needed by an app to provide its core functionalities before installation.

Many approaches aim to offer a finer-grade control over permissions using a wide array of strategies [26], [49], [50], [51], [52]. In particular, Mockdroid [26], TISSA [51] and SHAMDROID [49] allow users to restrict access to a resource by providing mocked information in place of real one whenever access to the resource is attempted. The evaluation presented shows that the majority of apps are able to continue execution when injected with mocked information but their usefulness might be greatly reduced for the end user. In [50], Jeon et al. define some strategies for developing fine-grained variants of Android permissions and implement Dr. Android, a prototype tool that allows to retrofit existing Android apps to make use of fine-grained permissions.

Proposed by Nauman et al., Apex [52] allows a user to selectively grant permissions to apps based on runtime contextual information, such as the location of the device or the number of times a resource has been previously used. To our knowledge, AFP is the first approach that allows users to specify permissions on a per-feature basis.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

Android devices are seamlessly supported. Authors of [2] University of Alabama at Birmingham Online Masters in Management Information Systems, “The future of mobile application,” 2014. [Online]. Available: http://businessdegrees.uab.edu/resources/infographic/the-future-of-mobile-application/

InstaGuard [44] instead propose a new approach that allows for instant deployment of hot-patches for mobile devices. The kernel vulnerabilities identified in [43], [44], together with the related kernel-level considerations therein, would be of undoubted interest towards a native low-level integration of the AFP approach directly in the Android operating system.

Researchers have investigated the effectiveness of nudging, i.e., the act of assisting individuals’ privacy and security choices with soft paternalistic interventions that drive users toward more beneficial options [65]. Integrating such techniques into AFP can potentially be beneficial for end users.

In [66] Almuhimedi et al. report on a study that evaluates the benefits of giving users an app permission manager and sending them nudges intended to raise their awareness of the data collected. Nudges proved to be beneficial, with 95 percent of participants reassessing their permissions, and 58 percent of them restricting some. In our approach, the role of permissions manager is performed by the AFP App.

In [67], Balebako et al. performed a series of experiments to examine how timing impacts the salience of smartphone app privacy notices. Notices displayed during app use showed significantly increased users’ recall rates over notices displayed in the app store. Liu et al. [68] implemented and evaluated a Personalized Privacy Assistant (PPA) to automatically recommend permissions to users based on prior answers to a small set of questions. In their experiments, 78.7 percent of the recommendations were adopted by users and the PPA was perceived as useful and usable. Introducing a similar assistant in the AFP App can ease the permission configuration process for end users.

# 8 CONCLUSIONS

In this paper, we have presented an approach aimed at overcoming some limitations of the current Android permission model. At the core of the proposed approach lies a new flexible permission model for Android apps in which end users can grant and negotiate the level of each single permission of a mobile app, on a per feature-basis.

The new permission model is supported by an infrastructure comprising three main components: a library internal to the apps enacting the approach at runtime; a stand-alone mobile app that allows end users to configure and negotiate at any time the permissions for each app on their devices; a web-based server for allowing developers to analyze their own mobile apps, and enhance them with the new flexible permission system with very low effort.

Evaluation of proposed approach has been conducted by designing, conducting, and reporting on four independent experiments aimed at empirically investigating key aspects of it: performance of the instrumenter, performance at runtime of instrumented apps, usability and acceptance of the approach for both end users and developers. Results are promising and provide confidence in the approach.

# REFERENCES

|[1]|A. Lella, A. Lipsman, and B. Martin, “The global mobile report: How multi-platform audiences and engagement compare in the US, Canada, UK and beyond,” 2015, comsCore white paper.|
|---|---|
|[2]|University of Alabama at Birmingham Online Masters in Management Information Systems, “The future of mobile application,” 2014. [Online]. Available: http://businessdegrees.uab.edu/resources/infographic/the-future-of-mobile-application/|
|[3]|A. P. Felt, E. Ha, S. Egelman, A. Haney, E. Chin, and D. Wagner, “Android permissions: User attention, comprehension, and behavior,” in Proc. 8th Symp. Usable Privacy Secur., 2012, pp. 3:1–3:14.|
|[4]|S. Chitkara, N. Gothoskar, S. Harish, J. I. Hong, and Y. Agarwal, “Does this app really need my location?: Context-aware privacy management for smartphones,” Proc. ACM Interactive Mobile Wearable Ubiquitous Technol., vol. 1, no. 3, 2017, Art. no. 42.|
|[5]|A. Razaghpanah, R. Nithyanand, N. Vallina-Rodriguez, S. Sundaresan, M. Allman, C. Kreibich, and P. Gill, “Apps, trackers, privacy, and regulators: A global study of the mobile tracking ecosystem,” 25th Annu. Net. Distrib. Syst. Secur. Symp., San Diego, California, USA, Feb. 18-21, 2018. [Online]. Available: http://wp.internetsociety.org/ndss/wp-content/uploads/sites/25/2018/02/ndss2018_05B-3_Razaghpanah_paper.pdf|
|[6]|M. E. Kummer and P. Schulte, “When private information settles the bill: Money and privacy in Google’s market for smartphone applications,” Manage. Sci. vol. 65, no. 8, pp. 3470–3494, 2019. [Online]. Available: https://doi.org/10.1287/mnsc.2018.3132, doi: 10.1287/mnsc.2018.3132.|
|[7]|A. P. F. Serge Egelman and D. Wagner, “Choice architecture and smartphone privacy: There’s a price for that,” in The Economics of Information Security and Privacy. Berlin, Germany: Springer, 2013, pp. 211–236.|
|[8]|G. L. Scoccia, S. Ruberto, I. Malavolta, M. Autili, and P. Inverardi, “An investigation into android run-time permissions from the end users’ perspective,” in Proc. 5th Int. Conf. Mobile Softw. Eng. Syst., MOBILESoft@ICSE, Gothenburg, Sweden, May 27-28, 2018, pp. 45–55. [Online]. Available: https://doi.org/10.1145/3197231.3197236, doi: 10.1145/3197231.3197236.|
|[9]|Z. Fang, W. Han, and Y. Li, “Permission based android security: Issues and countermeasures,” Comput. Secur., vol. 43, pp. 205–218, 2014.|
|[10]|Y. Zhauniarovich and O. Gadyatskaya, “Small changes, big changes: An updated view on the android permission system,” in Proc. Int. Symp. Res. Attacks Intrusions Defenses, 2016, pp. 346–367.|
|[11]|M. Madden, L. Rainie, K. Zickuhr, M. Duggan, and A. Smith, “Public perceptions of privacy and security in the post-snowden era,” Pew Res. Center, vol. 12, 2014.|
|[12]|ICT Leit Work Programme 2018–2020, 13 November 2018, 2018. [Online]. Available: http://ec.europa.eu/research/participants/data/ref/h2020/wp/2018-2020/main/h2020-wp1820-leit-ict_en.pdf|
|[13]|P. Bourque, R. E. Fairley, et al., Guide to the Software Engineering Body of Knowledge (SWEBOK (R)): Version 3.0. Los Alamitos, CA, USA: IEEE Comput. Soc. Press, 2014.|
|[14]|G. L. Scoccia, I. Malavolta, M. Autili, A. Di Salle, and P. Inverardi, “User-centric android flexible permissions,” in Proc. IEEE/ACM 39th Int. Conf. Softw. Eng. Companion, 2017, pp. 365–367.|
|[15]|P. G. Kelley, L. F. Cranor, and N. Sadeh, “Privacy as part of the app decision-making process,” in Proc. SIGCHI Conf. Human Factors Comput. Syst., 2013, pp. 3393–3402.|
|[16]|P. G. Kelley, S. Consolvo, L. F. Cranor, J. Jung, N. Sadeh, and D. Wetherall, “A conundrum of permissions: Installing applications on an android smartphone,” in Proc. 16th Int. Conf. Financial Cryptography Data Secur., 2012, pp. 68–79.|
|[17]|P. Andriotis, G. Stringhini, and M. A. Sasse, “Studying users’ adaptation to android’s run-time fine-grained access control system,” J. Inf. Secur. Appl., vol. 40, pp. 31–43, 2018.|
|[18]|S. Motiee, K. Hawkey, and K. Beznosov, “Do windows users follow the principle of least privilege?: Investigating user account control practices,” in Proc. 6th Symp. Usable Privacy Secur., 2010, pp. 1:1–1:13.|
|[19]|D. Akhawe and A. P. Felt, “Alice in warningland: A large-scale field study of browser security warning effectiveness,” in Proc. USENIX Secur. Symp., 2013, pp. 257–272.|
|[20]|R. B€ohme and J. Grossklags, “The security cost of cheap user interaction,” in Proc. New Secur. Paradigms Workshop, 2011, pp. 67–82.|
|[21]|N. S. J. Lin, B. Liu, and J. I. Hong, “Modeling users’ mobile app privacy preferences: Restoring usability in a sea of permission settings,” in Proc. Symp. Usable Privacy Secur., 2014, pp. 199–212.|

# IEEE TRANSACTIONS ON SOFTWARE ENGINEERING, VOL. 47, NO. 10, OCTOBER 2021

# References

1. R. Vallee-Rai, P. Co, E. Gagnon, L. Hendren, P. Lam, and V. Sundaresan, “Soot - A Java bytecode optimization framework,” in Proc. Conf. Centre Adv. Stud. Collaborative Res., 1999, Art. no. 13.
2. A. Bartel, J. Klein, Y. Le Traon, and M. Monperrus, “Dexpler: Converting android Dalvik bytecode to Jimple for static analysis with soot,” in Proc. ACM SIGPLAN Int. Workshop State Art Java Program Anal., 2012, pp. 27–38.
3. Android developers. [Online]. Available: http://developer.android.com/training/permissions/requesting.html
4. Z. Fang, W. Han, D. Li, Z. Guo, D. Guo, X. S. Wang, Z. Qian, and H. Chen, “revDroid: Code analysis of the side effects after dynamic permission revocation of android apps,” in Proc. 11th ACM Asia Conf. Comput. Commun. Secur., 2016, pp. 747–758.
5. A. R. Beresford, A. Rice, N. Skehin, and R. Sohan, “MockDroid: Trading privacy for application functionality on smartphones,” in Proc. 12th Workshop Mobile Comput. Syst. Appl., 2011, pp. 49–54.
6. A. P. Felt, S. Egelman, and D. Wagner, “I’ve got 99 problems, but vibration ain’t one: A survey of smartphone users’ concerns,” in Proc. 2nd ACM Workshop Secur. Privacy Smartphones Mobile Devices, 2012, pp. 33–44.
7. P. H. Chia, Y. Yamamoto, and N. Asokan, “Is this app safe?: A large scale study on application permissions and risk signals,” in Proc. 21st Int. Conf. World Wide Web, 2012, pp. 311–320.
8. C. Wohlin, P. Runeson, M. H€ost, M. Ohlsson, B. Regnell, and A. Wesslen, Experimentation in Software Engineering. Berlin, Germany: Springer, 2012.
9. I. Malavolta, S. Ruberto, T. Soru, and V. Terragni, “Hybrid mobile apps in the Google play store: An exploratory investigation,” in Proc. 2nd Int. Conf. Mobile Softw. Eng. Syst., 2015, pp. 56–59.
10. I. Gartner, “Gartner says free apps will account for nearly 90 percent of total mobile app store downloads in 2012,” 2012. [Online]. Available: http://www.gartner.com/newsroom/id/2153215
11. T. Das, M. Di Penta, and I. Malavolta, “A quantitative and qualitative investigation of performance-related commits in android apps,” in Proc. IEEE Int. Conf. Softw. Maintenance Evolution, 2016, pp. 443–447.
12. J. Brooke, et al., “SUS-a quick and dirty usability scale,” in Usability Evaluation in Industry. London, U.K.: Taylor & Francis, 1996, pp. 189–194.
13. A. Bangor, P. T. Kortum, and J. T. Miller, “An empirical evaluation of the system usability scale,” Int. J. Human-Comput. Interac., vol. 24, no. 6, pp. 574–594, 2008.
14. J. Kirakowski and M. Corbett, “SUMI: The software usability measurement inventory,” Brit. J. Educational Technol., vol. 24, no. 3, pp. 210–212, 1993.
15. R. Balebako, A. Marsh, J. Lin, J. I. Hong, and L. F. Cranor, “The privacy and security behaviors of smartphone app developers,” in Proc. Workshop Usable Secur., USEC 2014, San Diego, CA, USA, Feb. 23, 2014.
16. E. Bodden, A. Sewe, J. Sinschek, H. Oueslati, and M. Mezini, “Taming reflection: Aiding static analysis in the presence of reflection and custom class loaders,” in Proc. 33rd Int. Conf. Softw. Eng., 2011, pp. 241–250.
17. A. Smith, “Smartphone ownership–2013 update,” Pew Res. Center: Washington DC, vol. 12, 2013, Art. no. 2013.
18. G. W. Corder and D. I. Foreman, Nonparametric Statistics: A Step-by-Step Approach. Hoboken, NJ, USA: Wiley, 2014.
19. R. Xu, H. Sa€ıdi, and R. Anderson, “Aurasium: Practical policy enforcement for android applications,” in Proc. 21st USENIX Secur. Symp., 2012, pp. 539–552.
20. Y. Wang, S. Hariharan, C. Zhao, J. Liu, and W. Du, “Compac: Enforce component-level access control in android,” in Proc. 4th ACM Conf. Data Appl. Secur. Privacy, 2014, pp. 25–36.
21. A. Dawoud and S. Bugiel, “DroidCap: OS support for capability-based permissions in android,” 26th Annu. Netw. Distrib. Syst. Secur. Symp., San Diego, California, USA, Feb. 24-27, 2019. [Online]. Available: https://www.ndss-symposium.org/ndss-paper/droidcap-os-support-for-capability-based-permissions-in-android/
22. Y. Chen, Y. Zhang, Z. Wang, L. Xia, C. Bao, and T. Wei, “Adaptive android kernel live patching,” in Proc. 26th Secur. Symp. USENIX, 2017, pp. 1253–1270.
23. Y. Chen, Y. Li, L. Lu, Y. Lin, H. Vijayakumar, Z. Wang, and X. Ou, “InstaGuard: Instantly deployable hot-patches for vulnerable system programs on android,” in Proc. 25th Annu. Netw. Distrib. Syst. Secur. Symp., 2018.
24. H. Hao, V. Singh, and W. Du, “On the effectiveness of api-level access control using bytecode rewriting in android,” in Proc. 8th ACM SIGSAC Symp. Inf. Comput. Commun. Secur., 2013, pp. 25–36.
25. M. Harman, Y. Jia, and Y. Zhang, “App store mining and analysis: MSR for app stores,” in Proc. 9th IEEE Work. Conf. Mining Softw. Repositories, 2012, pp. 108–111.
26. Y. Guo, Y. Li, Z. Yang, and X. Chen, “What’s inside my app?: Understanding feature redundancy in mobile apps,” in Proc. 26th Conf. Program Comprehension, 2018, pp. 266–276.
27. H. Wang, Y. Li, Y. Guo, Y. Agarwal, and J. I. Hong, “Understanding the purpose of permission use in mobile apps,” ACM Trans. Inf. Syst., vol. 35, no. 4, 2017, Art. no. 43.
28. A. Acquisti, I. Adjerid, R. Balebako, L. Brandimarte, L. F. Cranor, S. Komanduri, P. G. Leon, N. Sadeh, F. Schaub, M. Sleeper, et al., “Nudges for privacy and security: Understanding and assisting users’ choices online,” ACM Comput. Surveys, vol. 50, no. 3, 2017, Art. no. 44.
29. H. Almuhimedi, F. Schaub, N. Sadeh, I. Adjerid, A. Acquisti, J. Gluck, L. F. Cranor, and Y. Agarwal, “Your location has been shared 5,398 times!: A field study on mobile app privacy nudging,” in Proc. 33rd Annu. ACM Conf. Human Factors Comput. Syst., 2015, pp. 787–796.

# SCOCCIA ET AL.: ENHANCING TRUSTABILITY OF ANDROID APPLICATIONS VIA USER-CENTRIC FLEXIBLE PERMISSIONS

2051

# References

1. R. Balebako, F. Schaub, I. Adjerid, A. Acquisti, and L. Cranor, “The impact of timing on the salience of smartphone app privacy notices,” in Proc. 5th Annu. ACM CCS Workshop Secur. Privacy Smartphones Mobile Devices, 2015, pp. 63–74.
2. B. Liu, M. S. Andersen, F. Schaub, H. Almuhimedi, S. A. Zhang, N. Sadeh, Y. Agarwal, and A. Acquisti, “Follow my recommendations: A personalized privacy assistant for mobile app permissions,” in Proc. 12th Symp. Usable Privacy Secur., 2016, pp. 27–41.
3. L. Jedrzejczyk, B. A. Price, A. K. Bandara, and B. Nuseibeh, “On the impact of real-time feedback on users’ behaviour in mobile location-sharing applications,” in Proc. 6th Symp. Usable Privacy Secur., 2010, Art. no. 14.

# Authors

# Gian Luca Scoccia

Received the PhD degree from the Gran Sasso Science Institute, Italy, in 2019. Currently is a post-doc with the University of L’Aquila, Italy. His research focuses on software engineering, empirical software engineering, privacy and security of mobile apps, mining of software repositories, program analysis.

# Marco Autili

Received the PhD degree in computer science from the University of L’Aquila. He is an associate professor with the Department of Information Engineering, Computer Science, and Mathematics, University of L’Aquila. His research focuses on automated synthesis for composing distributed systems, context-oriented mobile software programming, resource-oriented analysis of mobile apps, formal specification, and checking of temporal properties.

# Amleto Di Salle

Received the PhD degree in computer science from the University of L’Aquila. He is research fellow with the Department of Information Engineering, Computer Science, and Mathematics, University of L’Aquila. His research focuses on application of software engineering methods and (practical) formalisms to the modeling, verification, analysis and automatic synthesis of component-based and service-oriented distributed systems. More information is available at http://people.disim.univaq.it/amletodisalle/.

# Paola Inverardi

Received the PhD (honorary) degree in computer science from Mälardalen University. She is a full professor with the Department of Information Engineering, Computer Science, and Mathematics, University of L’Aquila. Her research focuses on software specification and verification of concurrent and distributed systems, deduction systems, and software architectures.

# Ivano Malavolta

Received the PhD degree in computer science from the University of L’Aquila, in 2012. He is assistant professor with the Department of Computer Science, Vrije Universiteit Amsterdam, The Netherlands. His research focuses on data-driven software engineering, with a special emphasis on mobile software development, software architecture, model-driven engineering, robotics software. He is applying empirical methods to assess practices and trends in the field of software engineering. He authored several scientific articles in international journals and peer-reviewed international conferences proceedings. He is program committee member and reviewer of international conferences and journals in the software engineering field. He is a member of the ACM, IEEE, VERSEN, Amsterdam Young Academy, and Amsterdam Data Science. More information is available at http://www.ivanomalavolta.com.

Authorized licensed use limited to: Pontificia Universidad Javeriana. Downloaded on August 13, 2024 at 04:26:33 UTC from IEEE Xplore. Restrictions apply.