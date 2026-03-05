# Contributing to the Self-Balancing Robot & Gesture Controller

This project is built iteratively. The roadmap is defined internally from Phase 0 to Phase 8.

## Phases Roadmap
- **Phase 0:** Dummy Load Tests & Hardware Validation
- **Phase 1:** FreeRTOS Scaffolding & I2C Testing
- **Phase 2:** MPU6050 Complementary Filter Implementation
- **Phase 3:** L298N PWM Control & Encoders
- **Phase 4:** PID Tuning & WebSockets Dashboard
- **Phase 5:** Edge Impulse Data Collection
- **Phase 6:** TFLite Micro Inference Task
- **Phase 7:** ESP-NOW Integration
- **Phase 8:** OTA Updates & Robustness Tests

## Guidelines
- Follow the engineering constraints (e.g., I2C Mutex, OTA safety) listed in the `README.md`.
- Test PID loops with the dashboard before merging to the `main` branch.
- Maintain the 96KB tensor arena constraint when modifying the TinyML gestures pipeline.
