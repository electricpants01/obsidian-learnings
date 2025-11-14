# Reinforcement Learning - Aprendizaje por Refuerzo

## Descripción
RL entrena agentes mediante recompensas: agent interactúa con environment, aprende policy óptima. Algoritmos: Q-learning, DQN, PPO, A3C. Casos: games (AlphaGo), robotics, recommendation systems, autonomous vehicles. Libraries: Gym, Stable-Baselines3, Ray RLlib.

## Ejemplo
```python
import gym
from stable_baselines3 import PPO
env = gym.make('CartPole-v1')
model = PPO('MlpPolicy', env, verbose=1)
model.learn(total_timesteps=10000)
```
---
**Tiempo**: 4-6 semanas
