<!--
 * @Author: Xuan 917125317@qq.com
 * @Date: 2022-06-13 23:44:51
 * @LastEditors: Xuan 917125317@qq.com
 * @LastEditTime: 2022-06-14 00:25:47
 * @FilePath: \tcyily.github.io\_posts\utils\2022-6-13-KinematicCharacterController.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# KinematicCharacterController
小黄人运动组件学习

# 系统核心代码
## KinematicCharacterSystem
该运动系统的管理类。是一个继承了MonoBehaviour的单例。有着下面的作用

1. 管理KinematicCharacterMotor和PhysicsMover的注册，数量限制Capacity。
2. 通过FixUpdate来进行运动前Position以及Rotation的设置。
3. LateUpdate中插值
4. 设置各个节点运动后的位置

        private void FixedUpdate()
        {
            if (Settings.AutoSimulation)
            {
                float deltaTime = Time.deltaTime;

                if (Settings.Interpolate) 
                {
                    PreSimulationInterpolationUpdate(deltaTime); // 初始化未移动时的Rotation、Position
                }

                Simulate(deltaTime, CharacterMotors, PhysicsMovers); // 移动

                if (Settings.Interpolate)
                {
                    PostSimulationInterpolationUpdate(deltaTime); // 设置运动后的Rotation、Position
                }
            }
        }

## KinematicCharacterMotor


## ICharacterController
移动相关的数据类。