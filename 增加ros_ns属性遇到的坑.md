---
    author: kresnikwang
    comments: true
    date: 2015-04-28 17:42:32+00:00
    layout: post
    title: PHP, Angular JS Development|My Export Quote|农产品出口工具开发
    categories:
    - Works
    - Tech
    tags:
    - bootstrap
    - javascript
    - php
    - AngularJS
    ---

#增加ros namespace属性中遇到的坑

'''

    <launch>    
        <!--- Sim Time -->
        <param name="/use_sim_time" value="true" />
    
        <!--- Run Rviz-->
        <node pkg="rviz" type="rviz" name="rviz" args="-d $(find lego_loam)/launch/test.rviz" />
        
        <group ns="robot0">
        <!--- TF -->
        <node pkg="tf" type="static_transform_publisher" name="camera_init_to_map"  args="0 0 0 1.570795   0        1.570795 /map    /camera_init 10" />
        <node pkg="tf" type="static_transform_publisher" name="base_link_to_camera" args="0 0 0 -1.570795 -1.570795 0        /camera /base_link   10" />
    
        <!--- LeGO-LOAM -->    
        <node pkg="lego_loam" type="imageProjection"    name="imageProjection"    output="screen"/>
        <node pkg="lego_loam" type="featureAssociation" name="featureAssociation" output="screen"/>
        <node pkg="lego_loam" type="mapOptmization"     name="mapOptmization"     output="screen"/>
        <node pkg="lego_loam" type="transformFusion"    name="transformFusion"    output="screen"/>
        
        </group>
        
          <node pkg="rosbag" type="play" name="playbag" args="--clock /home/xyt/论文数据/library/multi/2018-10-24-22-37-56.bag /velodyne_points:=/robot0/velodyne_points"/>
    
    </launch>
    
'''

******
###命名空间的作用范围 
1.图资源

   + Nodes: A node is an executable that uses ROS to communicate with other nodes.
   + Messages: ROS data type used when subscribing or publishing to a topic.
   + Topics: Nodes can publish messages to a topic as well as subscribe to a topic to receive messages.
   + Master: Name service for ROS (i.e. helps nodes find each other)
   + rosout: ROS equivalent of stdout/stderr
   + roscore: Master + rosout + parameter server (parameter server will be introduced later)

2.名称解析

   + 相对名称：相对名称的解析是依赖默认命名空间的。如默认命名空间为”/” 则名称“A/B”被解析为”/A/B”。
   + 基本名称：基本名称是没有命名空间限定符的相对名称（即没有/号）。
   + 全局名称：以“/”开头的名称称为全局名称，代表该名称属于全局命名空间。意思是在ROS系统的任何地方都可以使用。无论在ROS系统的任何地方它都以明确的意义。
   + 私有名称：私有名称以”~”开头，它与相对图名称的区别是，它的解析不依赖与默认命名空间，而是依赖包
   
3.因而命名空间仅作用于命名为相对名称的图资源上，其中命名空间通常由ros::nodeHandle nh("ns")指定。故而针对需求，我要把代码中的绝对名称命名的topic(/xx)前面的/去掉，将其改为相对名称，并且把构造函数中nh("~")删掉。这样修改后，ns属性才能真正作用于topic名称上。

******
###/use_sim_time参数
1. /use_sim_time设置为true,则系统使用仿真时间，ros::time()返回包内clock时间，当然前提是播包时要设置--clock(播放clock话题)。
2. tf关系是带stamp的，/use_sime_time=false,则static_transform_publisher播的tf的时间戳是walltime和包里数据的时间不一致，则会tf关系缺失
3. 把rosbag放到group里面，clock变成/ns/clock,则订阅/clock的收不到clock。
4. rosbag pub 的是 clock，而其他节点 sub 的是/clock？？？
