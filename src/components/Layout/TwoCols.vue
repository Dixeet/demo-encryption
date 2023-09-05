<template>
  <v-container>
    <slot></slot>
    <v-row>
      <v-col cols="6">
        <v-card>
          <v-card-text>
            <slot name="left"></slot>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="6">
        <v-card>
          <v-card-text>
            <slot name="right"></slot>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script setup></script>
