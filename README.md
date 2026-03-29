# Smart-Voice-Assistant-OpenEnv
Custom OpenEnv environment with voice-guided navigation for the visually impaired.
import numpy as np

class SmartBlindAssistant:
    def __init__(self):
        self.reset()

    def reset(self):
        self.agent_pos = [0, 0]
        self.goal_pos = [4, 4]
        # This is the "POST OK" part the system wants:
        return {"agent": self.agent_pos, "goal": self.goal_pos}

    def step(self, action):
        # Move logic
        if action == "up": self.agent_pos[1] += 1
        elif action == "down": self.agent_pos[1] -= 1
        elif action == "left": self.agent_pos[0] -= 1
        elif action == "right": self.agent_pos[0] += 1
        
        done = self.agent_pos == self.goal_pos
        reward = 100 if done else -1
        return {"agent": self.agent_pos}, reward, done, {}

# This is what the Scaler system calls
def run_inference():
    env = SmartBlindAssistant()
    return env.reset()

if __name__ == "__main__":
    run_inference()
    import numpy as np
from IPython.display import Javascript, display

# 🔊 Voice function (works in Google Colab)
def speak(text):
    display(Javascript(f'speechSynthesis.speak(new SpeechSynthesisUtterance("{text}"))'))
    print(f"🔊 AI Assistant: {text}")

class SmartBlindAssistant:
    def __init__(self):
        self.grid_size = 5
        self.reset()

    def reset(self):
        self.agent_pos = np.array([0, 0])
        self.goal_pos = np.array([4, 4])
        self.obstacles = [np.array([1, 1]), np.array([2, 2]), np.array([3, 3])]
        self.steps = 0
        self.score = 0

    def display_grid(self):
        grid = [["." for _ in range(self.grid_size)] for _ in range(self.grid_size)]

        for obs in self.obstacles:
            grid[obs[1]][obs[0]] = "X"

        grid[self.goal_pos[1]][self.goal_pos[0]] = "G"
        grid[self.agent_pos[1]][self.agent_pos[0]] = "A"

        print("\n📍 CURRENT MAP (A=Agent, G=Goal, X=Obstacle):")
        for row in reversed(grid):
            print(" ".join(row))

    def step(self, action):
        prev_pos = self.agent_pos.copy()

        # Movement logic
        if action == "up" and self.agent_pos[1] < self.grid_size - 1:
            self.agent_pos[1] += 1
        elif action == "down" and self.agent_pos[1] > 0:
            self.agent_pos[1] -= 1
        elif action == "left" and self.agent_pos[0] > 0:
            self.agent_pos[0] -= 1
        elif action == "right" and self.agent_pos[0] < self.grid_size - 1:
            self.agent_pos[0] += 1
        else:
            speak("Invalid move")
            return False

        self.steps += 1
        reward = -1  # Step penalty

        # ❌ Obstacle
        if any(np.array_equal(self.agent_pos, o) for o in self.obstacles):
            self.agent_pos = prev_pos
            reward = -10
            speak("⚠️ Obstacle ahead! Try another direction.")

        # 🎯 Goal
        elif np.array_equal(self.agent_pos, self.goal_pos):
            reward = 100
            self.score += reward
            self.display_grid()
            speak("🎉 Congratulations! You reached your destination.")
            print(f"\n🏆 Final Score: {self.score}")
            return True

        # Update score
        self.score += reward

        # 🧠 Guidance
        self.guide()

        print(f"⭐ Current Score: {self.score}")
        return False

    def guide(self):
        ax, ay = self.agent_pos
        gx, gy = self.goal_pos

        if ax < gx:
            speak("Move right")
        elif ax > gx:
            speak("Move left")
        elif ay < gy:
            speak("Move up")
        elif ay > gy:
            speak("Move down")

# 🚀 RUN PROGRAM
env = SmartBlindAssistant()
speak("Smart Voice Navigation System Started")

while True:
    env.display_grid()
    move = input("\nEnter move (up/down/left/right) or 'q' to quit: ").strip().lower()

    if move == 'q':
        speak("Program stopped")
        break

    done = env.step(move)

    if done:
        break
