# Assumed setup: motors are connected to ports B, C, and E.
# The other hub must broadcast a game number and a result (0 or 1) on channel "!O".
from spikecomm import Scanner
from hub import port
import runloop
import motor

# Change these values to set the motor turns and pause time.
E_TURN_DEGREES = 3600
B_TURN_DEGREES = 17 * 360
MOTOR_SPEED = 360
E_PAUSE_MS = 5000

# This stores the newest broadcast. None means nothing has arrived yet.
latest = {"game": None, "result": None}

def on_broadcast(values):
    # Save the game number and the result, but only if both values arrived.
    if len(values) == 2:
        latest["game"] = values[0]
        latest["result"] = values[1]

# Listen on the same channel that the other hub broadcasts on.
scanner = Scanner("!O")
scanner.listen(on_broadcast)

async def main():
    # Wait until we hear the game hub for the first time.
    print("Waiting for the game hub...")
    while latest["game"] is None:
        await runloop.sleep_ms(20)

    # The game hub is still showing its last game, which is old news.
    # Remember its number so we don't act on it.
    last_game = latest["game"]

    # Keep reacting to new games forever.
    while True:
        # Wait until the game number changes: that means a new game just ended.
        print("Waiting for the next game to finish...")
        while latest["game"] == last_game:
            await runloop.sleep_ms(20)

        # Remember this game number so we only act on this game once.
        last_game = latest["game"]
        received_option = latest["result"]
        print("Game", last_game, "result:", received_option)

        if received_option == 0:
            # Option 0: turn motor E, pause, then turn it the opposite way.
            await motor.run_for_degrees(port.E, E_TURN_DEGREES, -1000)
            await runloop.sleep_ms(E_PAUSE_MS)
            await motor.run_for_degrees(port.E, E_TURN_DEGREES, 1000)

        elif received_option == 1:
            # Option 1: turn motor C, reverse it, then turn motor B.
            await motor.run_for_degrees(port.C, 360, MOTOR_SPEED)
            await runloop.sleep_ms(4000)
            await motor.run_for_degrees(port.C, 360, -MOTOR_SPEED)
            await runloop.sleep_ms(1000)
            await motor.run_for_degrees(port.B, B_TURN_DEGREES, 1000)

runloop.run(main())
