Возникает проблема с QoS при подписке на навигацию из топика /mavros/local_position/pose

```bash
[WARN] [1687519570.979874421] [position_regulator_node]: New publisher discovered on topic '/uav_1/local_position/pose', offering incompatible QoS. No messages will be sent to it. Last incompatible policy: RELIABILITY_QOS_POLICY
```

Данную проблему удалось решить переназначением стандартного QoS:
```cpp
auto sensor_qos = rclcpp::QoS(rclcpp::SensorDataQoS());
// Subscribers
local_pose_sub = node->create_subscription<geometry_msgs::msg::PoseStamped>("/uav_1/local_position/pose", sensor_qos, std::bind(&PositionalRegulator::LocalPoseCb, this, std::placeholders::_1));
```