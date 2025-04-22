# ModDanishLearning

name: Build Mod
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Java 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Cache Gradle
        uses: actions/cache@v3
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
      - name: Build
        run: ./gradlew build --no-daemon
      - name: Upload jar
        uses: actions/upload-artifact@v3
        with:
          name: ModDanishLearning
          path: build/libs/ModDanishLearning-1.0.0.jar

          group = 'com.example'
version = '1.0.0'
archivesBaseName = 'ModDanishLearning'

plugins {
    id 'net.minecraftforge.gradle' version '5.1.16'
    id 'java'
}

sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenCentral()
}

dependencies {
    minecraft 'net.minecraftforge:forge:1.20.4-47.1.0'
}

minecraft {
    mappings channel: 'official', version: '1.20.4'
    runs {
        client {
            workingDirectory project.file('run')
            mods {
                danishlearning {
                    source sourceSets.main
                }
            }
        }
    }
}

rootProject.name = 'ModDanishLearning'

modLoader = 'javafml'
loaderVersion = '[47,)'
license = 'MIT'

[[mods]]
modId = 'danishlearning'
version = '1.0.0'
displayName = 'Danish Learning Mod'
author = 'Tuonome'
description = '''
Una mod per Minecraft che ogni giorno invia una parola danese con traduzione.
'''

{
  "itemGroup.danishlearning": "Learn Danish",
  "danishlearning.word_of_the_day": "Word of the day",
  "danishlearning.quiz_question": "Quiz: How do you say '%s' in Danish?"
}

package com.example.danishlearning;

import net.minecraftforge.fml.common.Mod;
import net.minecraftforge.eventbus.api.IEventBus;
import net.minecraftforge.fml.javafmlmod.FMLJavaModLoadingContext;

@Mod("danishlearning")
public class ModDanishLearning {
    public ModDanishLearning() {
        IEventBus bus = FMLJavaModLoadingContext.get().getModEventBus();
        EventHandlers.register(bus);
    }
}

package com.example.danishlearning;

import net.minecraftforge.event.TickEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;
import net.minecraft.server.level.ServerPlayer;
import net.minecraft.network.chat.Component;

import java.time.LocalDate;
import java.util.HashMap;
import java.util.Map;

@Mod.EventBusSubscriber(modid = "danishlearning")
public class EventHandlers {
    private static final Map<LocalDate, String[]> WORDS = new HashMap<>();
    static {
        WORDS.put(LocalDate.of(2025, 4, 22), new String[]{"Hej", "Ciao"});
        WORDS.put(LocalDate.of(2025, 4, 23), new String[]{"Tak", "Grazie"});
        // Aggiungi altre date e parole qui
    }
    private static LocalDate lastSent = null;

    public static void register(IEventBus bus) {
        bus.addListener(EventHandlers::onServerTick);
    }

    public static void onServerTick(TickEvent.ServerTickEvent event) {
        LocalDate today = LocalDate.now();
        if (event.phase == TickEvent.Phase.END && !today.equals(lastSent)) {
            if (WORDS.containsKey(today)) {
                String[] pair = WORDS.get(today);
                for (ServerPlayer player : event.getServer().getPlayerList().getPlayers()) {
                    player.sendMessage(Component.literal("🇩🇰 " + pair[0] + " = " + pair[1]), player.getUUID());
                }
            }
            lastSent = today;
        }
    }
}

