# micro-ROS Firmware Demo

This repository contains a demo firmware project using micro-ROS, designed for enimia AI edition car  . The firmware is compatible with STM32 and integrates essential micro-ROS features.

---

## Features

- **Platform Support**: Compatible with STM32F4 Discovery board.
- **micro-ROS Integration**: Includes micro-ROS client for seamless communication with ROS 2 nodes.
- **Peripheral Support**: CDC USB communication, LEDs.
- **Extensibility**: Easily adaptable to different hardware platforms.

---

## Requirements

### Hardware
-  STM32F4 Discovery Board
-  USB cable 

### Software
- STM32CubeIDE 
- ROS 2 Humble
- micro-ros agent

---

## Getting Started

### 1. Clone the Repository
```bash
git clone https://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@github.com/MansourACH/micro-ros_Demo.git
cd micro-ros_Demo
```
## Code Explanation

The following code demonstrates the use of micro-ROS APIs to create a simple publisher and subscriber system. 

### Key Features:
1. **Publisher**: Publishes integer messages (`std_msgs/Int32`) to the topic `stm32_publisher`.
2. **Subscriber**: Subscribes to velocity commands (`geometry_msgs/Twist`) on the topic `cmd_vel`.
3. **Executor**: Manages the callback for the subscriber and ensures non-blocking execution.

### Code:
```c
// Include necessary libraries for ROS2 and micro-ROS
rcl_publisher_t publisher; // Publisher object
std_msgs__msg__Int32 msg;  // Message object for publishing an integer
rclc_support_t support;    // Support object for managing initialization
rcl_allocator_t allocator; // Allocator for memory management
rcl_node_t node;           // Node object representing the micro-ROS node
rcl_subscription_t subscriber; // Subscriber object
geometry_msgs__msg__Twist msg_sub; // Message object for receiving velocity commands
rclc_executor_t executor;  // Executor for managing callbacks

// Initialize the default allocator
allocator = rcl_get_default_allocator();

// Initialize the support structure with default options
rclc_support_init(&support, 0, NULL, &allocator);

// Create a micro-ROS node with the name "TPE_Publisher"
rclc_node_init_default(&node, "TPE_Publisher", "", &support);

// Initialize the publisher to send messages of type std_msgs/Int32 on the "stm32_publisher" topic
rclc_publisher_init_default(
    &publisher,
    &node,
    ROSIDL_GET_MSG_TYPE_SUPPORT(std_msgs, msg, Int32),
    "stm32_publisher"
);

// Initialize the subscriber to receive messages of type geometry_msgs/Twist on the "cmd_vel" topic
rclc_subscription_init_default(
    &subscriber,
    &node,
    ROSIDL_GET_MSG_TYPE_SUPPORT(geometry_msgs, msg, Twist),
    "cmd_vel"
);

// Initialize the executor to handle callbacks and link it to the context
rclc_executor_init(&executor, &support.context, 1, &allocator);

// Add the subscriber to the executor, linking it with a callback function
rclc_executor_add_subscription(&executor, &subscriber, &msg_sub, &subscription_callback, ON_NEW_DATA);

// Initialize the message data
msg.data = 0;

// Infinite loop for processing ROS2 communication
for (;;)
{
    // Spin the executor to process incoming subscriptions and callbacks
    rclc_executor_spin_some(&executor, RCL_MS_TO_NS(100));

    // Publish the message on the "stm32_publisher" topic
    rcl_ret_t ret = rcl_publish(&publisher, &msg, NULL);
    if (ret != RCL_RET_OK)
    {
        // Print an error message if publishing fails
        printf("Error publishing (line %d)\n", __LINE__);
    }

    // Increment the message data for the next publish cycle
    msg.data++;

    // Delay for 10 ms
    osDelay(10);
}

