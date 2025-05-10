# Day 19: I merely noticed that you’re improperly stored, my dear secret!

**Learning Objectives**
- Understand how to interact with an executable's API.
- Intercept and modify internal APIs using Frida.
- Hack a game with the help of Frida.

### Game Hacking
Even while penetration testing is becoming increasingly popular, game hacking only makes up a small portion of the larger cyber security field. With its 2023 revenue reaching approximately $183.9 billion, the game industry can easily attract attackers. They can do various malicious activities, such as providing illegitimate ways to activate a game, providing bots to automate game actions, or misusing the game logic to simplify it. Therefore, hacking a game can be pretty complex since it requires different skills, including memory management, reverse engineering, and networking knowledge if the game runs online.

### Executables and Libraries
The **executable** file of an application is generally understood as a standalone binary file containing the compiled code we want to run. While some applications contain all the code they need to run in their executables, many applications usually rely on external code in library files with the "so" extension.

Library files are collections of functions that many applications can reuse. Unlike applications, they can't be directly executed as they serve no purpose by themselves. For a library function to be run, an executable will need to call it. The main idea behind libraries is to pack commonly used functions so developers don't need to reimplement them for every new application they develop.

### Hacking with Frida
Frida is a powerful instrumentation tool that allows us to analyse, modify, and interact with running applications. How does it do that? Frida creates a thread in the target process; that thread will execute some bootstrap code that allows the interaction. This interaction, known as the agent, permits the injection of JavaScript code, controlling the application's behaviour in real-time. One of the most crucial functionalities of Frida is the Interceptor. This functionality lets us alter internal functions' input or output or observe their behaviour.

### TryUnlockMe - The Frostbitten OTP
1. Start the game by running the following command on a terminal: `cd /home/ubuntu/Desktop/TryUnlockMe && ./TryUnlockMe`

2. Exploring the game, you will find a penguin asking for a PIN. Terminate the previous game instance and execute the following Frida command to intercept all the functions in the `libaocgame.so` library where some of the game logic is present: `frida-trace ./TryUnlockMe -i 'libaocgame.so!*'`

3. If you revisit the game, you can trigger the OTP function on the console displayed as `set_otpi`

![image](https://github.com/user-attachments/assets/33029637-1d97-4876-807d-03690eaff4e6)

Notice the output `_Z7set_otpi` indicates that the `set_otp` function is called during the NPC interaction; you can try intercepting it!

4. Open a new terminal. Go to the `/home/ubuntu/Desktop/TryUnlockMe/__handlers__/libaocgame.so/` folder, and open Visual Studio Code by running:

![image](https://github.com/user-attachments/assets/b88a163f-2237-4c76-84d4-532d9a60d09c)

![image](https://github.com/user-attachments/assets/587a7854-125a-45c8-81c0-661fdd781562)

5. At this point, you should be able to select the `_Z7set_otpi` JavaScript file with the hook defined. The i at the end of the `set_otp` function indicates that an integer will be passed as a parameter. It will likely set the OTP by passing it as the first argument. To get the parameter value, you can use the `log` function, specifying the first elements of the array `args` on the `onEnter` function: `log("Parameter:" + args[0].toInt32());`. Now Save.

Your JavaScript file should look like the following:

![image](https://github.com/user-attachments/assets/07b6d0a7-befb-49ea-8786-57517647d801)

6. Return to the game until you reach this point:

![image](https://github.com/user-attachments/assets/7bf36ade-a8a2-43c8-bfd0-11f1262fef43)

7. Go back to the terminal which should show this:

![image](https://github.com/user-attachments/assets/72ba9e78-ccf0-4ec4-beaf-7d5be969bdb1)

8. Use the parameter (in this case 685686) as the OTP. This reveals the flag.

![image](https://github.com/user-attachments/assets/c1ef0d2d-5f10-45b1-837a-9ee0c2a1338e)

### TryUnlockMe - A Wishlist for Billionaires
1. Explore the new stage of the game. You will find another penguin with a costly item ("Flag 2") named Right of Pass. Try to buy it and you'll be told you don't have enough money. The game lets you earn coins by using the old PC on the field, but getting 1.000.000 coins that way sounds tedious. You can again use Frida to intercept the function in charge of purchasing the item.

2. This time is a bit more tricky than the previous one because the function `buy_item` displayed as: `_Z17validate_purchaseiii` has three `i` letters after its name to indicate that it has three integer parameters. You can log those values using the log function for each parameter trying to buy something:

`log("Parameter1:" + args[0].toInt32())`  
`log("Parameter2:" + args[1].toInt32())`  
`log("Parameter3:" + args[2].toInt32())`

3. Save the file. Your JavaScript `buy_item` file will look like this:

![image](https://github.com/user-attachments/assets/5f00b5b9-9995-46ca-98b3-f9e12c2460de)

4. Return to the game and try to buy the flag again. In the terminal you should be able to log something similar:

![image](https://github.com/user-attachments/assets/436cb515-f8dc-47d7-9f04-39c923fc1c20)

By simple inspection, we can determine that the first parameter is the Item ID, the second is the price, and the third is the player's coins.

5. Let's manipulate the price and set it as zero, so we can buy any item that we want. Insert `args[1] = ptr(0)` into the JavaScript `buy_item` file so it looks like this:

![image](https://github.com/user-attachments/assets/9bb8166f-de8e-4130-bbd3-3736fbb66898)

6. Return to the game and try to buy Flag 2. We can buy the item and the flag is revealed.

![image](https://github.com/user-attachments/assets/0aeb34f8-4fc2-471a-a824-19adfc8cf8dc)

### TryUnlockMe - Naughty Fingers, Nice Hack
1. Continue with the game and find the third penguin.
2. This is what shows in the terminal:

![image](https://github.com/user-attachments/assets/eeb8fdd3-f57a-4a10-9119-7088d5b5c989)

This last stage of the game is a bit more tricky because the output displayed by Frida is `_Z16check_biometricsPKc()`, so it does not handle integers anymore (notice there isn't an `i` at the end of it like `_Z17validate_purchaseiii` or `_Z7set_otpi`), but strings making a bit more complex to debug.

3. By selecting the JavaScript file named `_Z16check_biometricsPKc`, you can add the following code to the `onEnter()` function as you did previously to debug the content of the parameter. Save the file. It should look like this:

![image](https://github.com/user-attachments/assets/dec64a68-8e19-4f12-ac50-54e71dca1f96)

4. Return and interact with the game. This is what logs in the terminal:

![image](https://github.com/user-attachments/assets/6c219098-e342-4b8d-a496-517c0904ed05)

5. This output does not seem very helpful; you may have to consider another way. You can log the return value of the function by adding the following log instruction in the `onLeave` function:

![image](https://github.com/user-attachments/assets/9b7b2029-9009-4c4f-a5ca-754001d7672e)

6. Interact with the game again. This will show in the terminal:

![image](https://github.com/user-attachments/assets/3204c8a5-0f6f-46da-b631-2d5b5ba5dbe8)

So, the value returned is 0, which may indicate that it is a boolean flag set to False. Which value will set it to True? Can you trick the game into thinking the biometrics check worked?

7. The following instruction will set it the return value to True: `retval.replace(ptr(1))`. Add this to the `onLeave` function, and Save.

![image](https://github.com/user-attachments/assets/d102f445-33ea-4181-8e25-02a54b423648)

8. Return to the game... Success!

![image](https://github.com/user-attachments/assets/8dc86a06-23ff-4ba0-9fdc-104cb9533648)

9. Notice the terminal is now showing a return value of 1:

![image](https://github.com/user-attachments/assets/c443626a-ed93-43a1-8c15-acacf0bb075f)

### Answers
1. What is the OTP flag? **THM{one_tough_password}**.
2. What is the billionaire item flag? **THM{credit_card_undeclined}**.
3. What is the biometric flag? **THM{dont_smash_your_keyboard}**.
