package stunslam;

import com.mojang.blaze3d.platform.InputConstants;
import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.event.lifecycle.v1.ClientTickEvents;
import net.fabricmc.fabric.api.client.keymapping.v1.KeyMappingHelper;
import net.minecraft.client.KeyMapping;
import net.minecraft.client.Minecraft;
import net.minecraft.client.player.LocalPlayer;
import net.minecraft.resources.Identifier;
import net.minecraft.tags.ItemTags;
import net.minecraft.world.entity.Entity;
import net.minecraft.world.entity.player.Player;
import net.minecraft.world.item.Items;
import net.minecraft.world.phys.EntityHitResult;

/**
 * Stun Slam per Tastendruck (Standard: Mausrad druecken).
 *
 * Ablauf:
 *  1. Du haeltst die Axt und visierst einen Spieler an.
 *  2. Taste druecken -> Axt-Schlag (deaktiviert den Schild).
 *  3. Kurze Pause -> Wechsel auf den Mace in deiner Hotbar.
 *  4. Kurze Pause -> Mace-Schlag, danach zurueck auf die Axt.
 *
 * Das Springen machst du selbst: Loese die Taste in der Luft aus,
 * damit der Mace-Schlag den Smash-Schaden bekommt.
 */
public class StunSlamClient implements ClientModInitializer {

    // ===== EINSTELLUNGEN (hier kannst du spaeter Werte aendern) =====
    // Wartezeit in Ticks (20 Ticks = 1 Sekunde)
    private static final int TICKS_AXT_BIS_WECHSEL = 1;
    // 0 = Wechsel und Mace-Schlag im selben Tick
    private static final int TICKS_WECHSEL_BIS_MACE = 2;
    // ================================================================

    // Mausrad-Taste (GLFW_MOUSE_BUTTON_MIDDLE = 2)
    private static final int MOUSE_MIDDLE = 2;

    private static KeyMapping slamKey;

    // 0 = bereit, 1 = warte auf Wechsel zum Mace, 2 = warte auf Mace-Schlag
    private static int stage = 0;
    private static int timer = 0;
    private static int axeSlot = 0;
    private static int maceSlot = 0;
    private static Entity target = null;

    @Override
    public void onInitializeClient() {
        KeyMapping.Category category = KeyMapping.Category.register(
                Identifier.fromNamespaceAndPath("stunslam", "keys"));

        slamKey = KeyMappingHelper.registerKeyMapping(new KeyMapping(
                "key.stunslam.slam",
                InputConstants.Type.MOUSE,
                MOUSE_MIDDLE,
                category));

        ClientTickEvents.END_CLIENT_TICK.register(StunSlamClient::tick);
    }

    private static void tick(Minecraft mc) {
        LocalPlayer player = mc.player;
        if (player == null || mc.gameMode == null || mc.level == null) {
            stage = 0;
            return;
        }

        // Tastendruecke einsammeln
        boolean pressed = false;
        while (slamKey.consumeClick()) {
            pressed = true;
        }

        if (stage == 0) {
            if (pressed) {
                start(mc, player);
            }
            return;
        }

        // Ziel weg oder tot -> abbrechen und Axt zurueck
        if (target == null || !target.isAlive()) {
            finish(player);
            return;
        }

        timer--;
        if (timer > 0) {
            return;
        }

        if (stage == 1) {
            // Wechsel auf den Mace
            player.getInventory().setSelectedSlot(maceSlot);
            if (TICKS_WECHSEL_BIS_MACE <= 0) {
                // Mace-Schlag direkt im selben Tick wie der Wechsel
                mc.gameMode.attack(player, target);
                finish(player);
            } else {
                stage = 2;
                timer = TICKS_WECHSEL_BIS_MACE;
            }
        } else if (stage == 2) {
            // Mace-Schlag
            mc.gameMode.attack(player, target);
            finish(player);
        }
    }

    private static void start(Minecraft mc, LocalPlayer player) {
        if (!player.getMainHandItem().is(ItemTags.AXES)) {
            return; // keine Axt in der Hand
        }

        if (!(mc.hitResult instanceof EntityHitResult hit)
                || !(hit.getEntity() instanceof Player enemy)) {
            return; // kein Spieler anvisiert
        }

        int foundMace = -1;
        for (int i = 0; i < 9; i++) {
            if (player.getInventory().getItem(i).is(Items.MACE)) {
                foundMace = i;
                break;
            }
        }
        if (foundMace == -1) {
            return; // kein Mace in der Hotbar
        }

        axeSlot = player.getInventory().getSelectedSlot();
        maceSlot = foundMace;
        target = enemy;

        // Schritt 1: Axt-Schlag
        mc.gameMode.attack(player, target);

        stage = 1;
        timer = TICKS_AXT_BIS_WECHSEL;
    }

    private static void finish(LocalPlayer player) {
        player.getInventory().setSelectedSlot(axeSlot);
        stage = 0;
        target = null;
    }
}
