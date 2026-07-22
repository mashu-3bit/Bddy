Architecture & Project Structure
Here is the complete, scalable project structure for the Next.js (App Router) platform:
birthday-gift-platform/
├── public/
│   ├── fonts/
│   └── sounds/
├── src/
│   ├── app/
│   │   ├── api/                 # API Routes (AI generation, PDF export, etc.)
│   │   ├── dashboard/           # Creator Dashboard (Admin Panel)
│   │   ├── [slug]/              # Dynamic Recipient Birthday Web App
│   │   ├── layout.tsx           # Root Layout with Providers & Music context
│   │   └── page.tsx             # Landing Page / Builder Entry
│   ├── components/
│   │   ├── builder/             # Dashboard & Form Builders
│   │   ├── celebration/         # Confetti, Fireworks, Balloons, Effects
│   │   ├── games/               # Interactive Mini-Games (Quiz, Scratch, etc.)
│   │   ├── sections/            # Core View Sections (Timeline, Cake, Gallery, Letter)
│   │   └── ui/                  # Reusable Glassmorphic UI Elements
│   ├── context/
│   │   ├── AudioContext.tsx     # Persistent Music Player State
│   │   └── ThemeContext.tsx     # Theme Manager (Romantic, Luxury, Neon, etc.)
│   ├── hooks/                   # Custom hooks (microphone, confetti, etc.)
│   ├── lib/                     # Firebase/Supabase client, Cloudinary utils
│   └── types/                   # TypeScript interfaces
├── tailwind.config.ts
├── tsconfig.json
└── package.json

Core Configuration & Setup
package.json
{
  "name": "birthday-gift-platform",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@supabase/supabase-js": "^2.39.8",
    "canvas-confetti": "^1.9.2",
    "framer-motion": "^11.1.7",
    "gsap": "^3.12.5",
    "lucide-react": "^0.359.0",
    "next": "14.2.1",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-canvas-confetti": "^2.0.7",
    "canvas-confetti-typescript": "^1.9.3",
    "qrcode.react": "^3.1.0"
  },
  "devDependencies": {
    "@types/canvas-confetti": "^1.9.0",
    "@types/node": "^20.11.30",
    "@types/react": "^18.2.73",
    "@types/react-dom": "^18.2.23",
    "postcss": "^8.4.38",
    "tailwindcss": "^3.4.1",
    "typescript": "^5.4.3"
  }
}

tailwind.config.ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        glass: {
          light: "rgba(255, 255, 255, 0.1)",
          dark: "rgba(15, 23, 42, 0.6)",
          border: "rgba(255, 255, 255, 0.2)",
        },
      },
      fontFamily: {
        sans: ["var(--font-inter)", "sans-serif"],
        handwriting: ["var(--font-caveat)", "cursive"],
        serif: ["var(--font-playfair)", "serif"],
      },
      animation: {
        "pulse-slow": "pulse 4s cubic-bezier(0.4, 0, 0.6, 1) infinite",
        "float": "float 6s ease-in-out infinite",
      },
      keyframes: {
        float: {
          "0%, 100%": { transform: "translateY(0px)" },
          "50%": { transform: "translateY(-15px)" },
        },
      },
    },
  },
  plugins: [],
};
export default config;

TypeScript Definitions (src/types/index.ts)
export type ThemeMode = 'romantic' | 'elegant' | 'cute' | 'dark' | 'luxury' | 'minimal' | 'anime' | 'galaxy' | 'neon';

export interface MemoryItem {
  id: string;
  url: string;
  caption: string;
  date: string;
}

export interface TimelineItem {
  id: string;
  title: string;
  date: string;
  story: string;
  image?: string;
}

export interface GiftConfig {
  id: string;
  slug: string;
  recipientName: string;
  senderName: string;
  theme: ThemeMode;
  musicUrl: string;
  letterMessage: string;
  images: MemoryItem[];
  timeline: TimelineItem[];
  gamesEnabled: {
    quiz: boolean;
    balloonPop: boolean;
    memoryMatch: boolean;
    scratchCard: boolean;
    spinWheel: boolean;
  };
  quizQuestions?: { question: string; options: string[]; answer: number }[];
  secretPassword?: string;
  surpriseCoupon?: string;
  createdAt: string;
}

Core Systems & Context Providers
Persistent Music Player (src/context/AudioContext.tsx)
'use client';

import React, { createContext, useContext, useState, useEffect, useRef } from 'react';

interface AudioContextType {
  isPlaying: boolean;
  togglePlay: () => void;
  setTrack: (url: string) => void;
  volume: number;
  setVolume: (v: number) => void;
}

const AudioContext = createContext<AudioContextType | undefined>(undefined);

export const AudioProvider: React.FC<{ children: React.ReactNode; initialUrl?: string }> = ({ children, initialUrl }) => {
  const [isPlaying, setIsPlaying] = useState(false);
  const [volume, setVolume] = useState(0.5);
  const audioRef = useRef<HTMLAudioElement | null>(null);

  useEffect(() => {
    if (initialUrl) {
      audioRef.current = new Audio(initialUrl);
      audioRef.current.loop = true;
      audioRef.current.volume = volume;
    }
    return () => {
      audioRef.current?.pause();
    };
  }, [initialUrl]);

  useEffect(() => {
    if (audioRef.current) {
      audioRef.current.volume = volume;
    }
  }, [volume]);

  const togglePlay = () => {
    if (!audioRef.current) return;
    if (isPlaying) {
      audioRef.current.pause();
    } else {
      audioRef.current.play().catch(() => {});
    }
    setIsPlaying(!isPlaying);
  };

  const setTrack = (url: string) => {
    if (audioRef.current) {
      audioRef.current.pause();
    }
    audioRef.current = new Audio(url);
    audioRef.current.loop = true;
    audioRef.current.volume = volume;
    if (isPlaying) {
      audioRef.current.play().catch(() => {});
    }
  };

  return (
    <AudioContext.Provider value={{ isPlaying, togglePlay, setTrack, volume, setVolume }}>
      {children}
    </AudioContext.Provider>
  );
};

export const useAudio = () => {
  const context = useContext(AudioContext);
  if (!context) throw new Error('useAudio must be used within an AudioProvider');
  return context;
};

Cinematic Experience Components
1. Welcome Splash & Glassmorphism Opening (src/components/sections/WelcomeScreen.tsx)
'use client';

import React, { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { Sparkles, Heart, Volume2 } from 'lucide-react';
import { useAudio } from '@/context/AudioContext';

interface WelcomeScreenProps {
  recipientName: string;
  senderName: string;
  onOpen: () => void;
}

export const WelcomeScreen: React.FC<WelcomeScreenProps> = ({ recipientName, senderName, onOpen }) => {
  const [isOpen, setIsOpen] = useState(false);
  const { togglePlay } = useAudio();

  const handleOpenClick = () => {
    setIsOpen(true);
    togglePlay();
    setTimeout(onOpen, 1000);
  };

  return (
    <AnimatePresence>
      {!isOpen && (
        <motion.div 
          initial={{ opacity: 1 }}
          exit={{ opacity: 0, scale: 1.1 }}
          transition={{ duration: 0.8 }}
          className="fixed inset-0 z-50 flex items-center justify-center bg-gradient-to-br from-purple-950 via-slate-900 to-black p-4 overflow-hidden"
        >
          {/* Ambient Floating Particles */}
          <div className="absolute inset-0 pointer-events-none">
            {[...Array(20)].map((_, i) => (
              <motion.div
                key={i}
                className="absolute bg-white/20 rounded-full blur-[1px]"
                style={{
                  width: Math.random() * 6 + 2,
                  height: Math.random() * 6 + 2,
                  top: `${Math.random() * 100}%`,
                  left: `${Math.random() * 100}%`,
                }}
                animate={{ y: [0, -30, 0], opacity: [0.2, 0.8, 0.2] }}
                transition={{ duration: 3 + Math.random() * 3, repeat: Infinity, ease: "easeInOut" }}
              />
            ))}
          </div>

          {/* Luxury Glass Card */}
          <motion.div 
            initial={{ scale: 0.9, opacity: 0, y: 20 }}
            animate={{ scale: 1, opacity: 1, y: 0 }}
            transition={{ duration: 0.6, delay: 0.2 }}
            className="relative z-10 max-w-lg w-full p-8 md:p-12 rounded-3xl bg-white/10 backdrop-blur-xl border border-white/20 shadow-[0_8px_32px_0_rgba(0,0,0,0.37)] text-center text-white"
          >
            <div className="mx-auto w-16 h-16 mb-6 rounded-full bg-gradient-to-tr from-pink-500 to-purple-500 flex items-center justify-center shadow-lg shadow-pink-500/30">
              <Sparkles className="w-8 h-8 text-white animate-spin-slow" />
            </div>

            <span className="text-sm uppercase tracking-widest text-pink-300 font-semibold">Special Birthday Gift</span>
            <h1 className="text-4xl md:text-5xl font-extrabold mt-2 mb-4 font-serif bg-gradient-to-r from-white via-pink-100 to-pink-300 bg-clip-text text-transparent">
              For {recipientName}
            </h1>
            <p className="text-slate-300 text-sm md:text-base mb-8">
              A cinematic digital journey crafted with love by <span className="text-pink-400 font-medium">{senderName}</span>.
            </p>

            <motion.button
              whileHover={{ scale: 1.05 }}
              whileTap={{ scale: 0.95 }}
              onClick={handleOpenClick}
              className="w-full py-4 px-8 rounded-2xl bg-gradient-to-r from-pink-500 via-purple-500 to-indigo-500 text-white font-bold shadow-lg shadow-purple-500/40 flex items-center justify-center gap-3 text-lg border border-white/20 transition-all"
            >
              <Heart className="w-5 h-5 text-white fill-white animate-bounce" />
              Open Your Surprise
            </motion.button>
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
};

2. Polaroid Memory Gallery (src/components/sections/MemoryGallery.tsx)
'use client';

import React, { useState } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { MemoryItem } from '@/types';
import { X, ZoomIn } from 'lucide-react';

export const MemoryGallery: React.FC<{ images: MemoryItem[] }> = ({ images }) => {
  const [selectedImage, setSelectedImage] = useState<MemoryItem | null>(null);

  return (
    <section className="py-20 px-4 max-w-7xl mx-auto">
      <div className="text-center mb-12">
        <h2 className="text-3xl md:text-4xl font-serif font-bold text-white mb-3">Our Beautiful Memories</h2>
        <p className="text-slate-400">Captured moments filled with joy, laughter, and timeless love.</p>
      </div>

      <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-8">
        {images.map((img, index) => (
          <motion.div
            key={img.id || index}
            initial={{ opacity: 0, y: 20 }}
            whileInView={{ opacity: 1, y: 0 }}
            viewport={{ once: true }}
            transition={{ delay: index * 0.1 }}
            whileHover={{ scale: 1.03, rotate: index % 2 === 0 ? 1 : -1 }}
            onClick={() => setSelectedImage(img)}
            className="bg-white/90 p-4 pb-6 rounded-lg shadow-xl cursor-pointer backdrop-blur-sm border border-white/20 transform transition-all"
          >
            <div className="relative aspect-square overflow-hidden rounded-md mb-4 bg-slate-100">
              <img src={img.url} alt={img.caption} className="object-cover w-full h-full transition-transform duration-500 hover:scale-110" />
              <div className="absolute inset-0 bg-black/20 opacity-0 hover:opacity-100 transition-opacity flex items-center justify-center">
                <ZoomIn className="text-white w-8 h-8" />
              </div>
            </div>
            <p className="font-handwriting text-slate-800 text-lg text-center truncate">{img.caption}</p>
            <span className="block text-center text-xs text-slate-400 mt-1">{img.date}</span>
          </motion.div>
        ))}
      </div>

      {/* Fullscreen Lightbox Modal */}
      <AnimatePresence>
        {selectedImage && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={() => setSelectedImage(null)}
            className="fixed inset-0 z-50 bg-black/90 backdrop-blur-md flex items-center justify-center p-4"
          >
            <button className="absolute top-6 right-6 text-white bg-white/10 p-3 rounded-full hover:bg-white/20 transition-all">
              <X className="w-6 h-6" />
            </button>
            <motion.div 
              initial={{ scale: 0.9 }}
              animate={{ scale: 1 }}
              exit={{ scale: 0.9 }}
              className="max-w-3xl w-full bg-white p-4 rounded-2xl shadow-2xl overflow-hidden"
              onClick={(e) => e.stopPropagation()}
            >
              <img src={selectedImage.url} alt={selectedImage.caption} className="w-full max-h-[70vh] object-contain rounded-lg mb-4" />
              <div className="text-center">
                <h3 className="font-handwriting text-2xl text-slate-800">{selectedImage.caption}</h3>
                <p className="text-sm text-slate-500 mt-1">{selectedImage.date}</p>
              </div>
            </motion.div>
          </motion.div>
        )}
      </AnimatePresence>
    </section>
  );
};

3. Interactive Birthday Cake with Microphone Candle Blow (src/components/sections/BirthdayCake.tsx)
'use client';

import React, { useState, useEffect } from 'react';
import { motion } from 'framer-motion';
import { Flame, Mic, Sparkles } from 'lucide-react';
import confetti from 'canvas-confetti';

export const BirthdayCake: React.FC = () => {
  const [candlesLit, setCandlesLit] = useState(true);
  const [isListening, setIsListening] = useState(false);

  const blowCandles = () => {
    if (!candlesLit) return;
    setCandlesLit(false);
    confetti({
      particleCount: 150,
      spread: 80,
      origin: { y: 0.6 }
    });
  };

  const startMicListener = async () => {
    try {
      setIsListening(true);
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)();
      const analyser = audioContext.createAnalyser();
      const microphone = audioContext.createMediaStreamSource(stream);
      microphone.connect(analyser);
      analyser.fftSize = 256;
      const bufferLength = analyser.frequencyBinCount;
      const dataArray = new Uint8Array(bufferLength);

      const checkAudio = () => {
        analyser.getByteFrequencyData(dataArray);
        let sum = 0;
        for (let i = 0; i < bufferLength; i++) {
          sum += dataArray[i];
        }
        const average = sum / bufferLength;

        if (average > 40) { // Threshold for blow detection
          blowCandles();
          stream.getTracks().forEach(track => track.stop());
          setIsListening(false);
        } else if (isListening) {
          requestAnimationFrame(checkAudio);
        }
      };
      checkAudio();
    } catch (err) {
      console.error("Microphone access denied or error:", err);
      setIsListening(false);
    }
  };

  return (
    <section className="py-20 flex flex-col items-center justify-center px-4">
      <div className="text-center mb-10">
        <h2 className="text-3xl md:text-4xl font-serif font-bold text-white mb-2">Make a Wish!</h2>
        <p className="text-slate-300 text-sm md:text-base">Click the button, tap the candles, or blow into your microphone!</p>
      </div>

      {/* Cake Container */}
      <div className="relative cursor-pointer" onClick={blowCandles}>
        {/* Flames */}
        <div className="flex justify-center gap-6 mb-2">
          {[1, 2, 3].map((_, i) => (
            <div key={i} className="relative flex flex-col items-center">
              {candlesLit && (
                <motion.div
                  animate={{ scale: [1, 1.2, 1], y: [0, -4, 0] }}
                  transition={{ repeat: Infinity, duration: 0.6, delay: i * 0.2 }}
                >
                  <Flame className="w-8 h-8 text-amber-400 fill-amber-300 filter drop-shadow-[0_0_10px_rgba(251,191,36,0.8)]" />
                </motion.div>
              )}
            </div>
          ))}
        </div>

        {/* Cake Graphic */}
        <div className="w-64 h-48 md:w-80 md:h-56 bg-gradient-to-b from-pink-300 via-pink-400 to-pink-500 rounded-t-3xl shadow-2xl relative border-4 border-white/20 flex flex-col items-center justify-end pb-6">
          {/* Icing Drips */}
          <div className="absolute top-0 inset-x-0 h-10 bg-white rounded-t-2xl shadow-inner flex justify-around">
            {[...Array(6)].map((_, i) => (
              <div key={i} className="w-6 h-6 bg-white rounded-b-full"></div>
            ))}
          </div>
          <div className="absolute inset-x-0 bottom-1/2 h-8 bg-pink-200/50 flex items-center justify-around">
            {[...Array(5)].map((_, i) => (
              <Sparkles key={i} className="w-4 h-4 text-white/70" />
            ))}
          </div>
          <span className="font-handwriting text-2xl font-bold text-white tracking-widest drop-shadow-md">Happy Birthday!</span>
        </div>
        <div className="w-72 md:w-96 h-8 bg-slate-300 rounded-full mx-auto shadow-2xl -mt-1 border border-white/40"></div>
      </div>

      {/* Controls */}
      <div className="flex flex-wrap gap-4 mt-8 justify-center">
        <motion.button
          whileHover={{ scale: 1.05 }}
          whileTap={{ scale: 0.95 }}
          onClick={blowCandles}
          className="px-6 py-3 rounded-2xl bg-gradient-to-r from-amber-400 to-orange-500 text-white font-bold shadow-lg shadow-orange-500/30 flex items-center gap-2"
        >
          <Flame className="w-5 h-5" />
          {candlesLit ? "Blow Candles" : "Relit Candles"}
        </motion.button>

        <motion.button
          whileHover={{ scale: 1.05 }}
          whileTap={{ scale: 0.95 }}
          onClick={startMicListener}
          disabled={isListening}
          className="px-6 py-3 rounded-2xl bg-gradient-to-r from-purple-500 to-indigo-500 text-white font-bold shadow-lg shadow-purple-500/30 flex items-center gap-2 disabled:opacity-50"
        >
          <Mic className="w-5 h-5" />
          {isListening ? "Listening for breath..." : "Blow via Mic"}
        </motion.button>
      </div>
    </section>
  );
};

Creator Dashboard & Builder (src/app/dashboard/page.tsx)
'use client';

import React, { useState } from 'react';
import { motion } from 'framer-motion';
import { GiftConfig, ThemeMode, MemoryItem, TimelineItem } from '@/types';
import { Sparkles, Image as ImageIcon, Music, Type, Calendar, Gamepad2, Share2, CheckCircle2 } from 'lucide-react';
import { QRCodeSVG } from 'qrcode.react';

export default function CreatorDashboard() {
  const [config, setConfig] = useState<GiftConfig>({
    id: 'gift-123',
    slug: 'sarah-2026',
    recipientName: 'Sarah',
    senderName: 'Alex',
    theme: 'romantic',
    musicUrl: 'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3',
    letterMessage: 'Happy Birthday to the most amazing person in the world! You bring endless joy to my life...',
    images: [
      { id: '1', url: 'https://images.unsplash.com/photo-1517841905240-472988babdf9', caption: 'Our first trip together!', date: 'June 2024' },
      { id: '2', url: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac', caption: 'Unforgettable sunset.', date: 'August 2025' }
    ],
    timeline: [
      { id: '1', title: 'First Met', date: 'January 12, 2023', story: 'We bumped into each other at the cozy downtown café.' }
    ],
    gamesEnabled: {
      quiz: true,
      balloonPop: true,
      memoryMatch: false,
      scratchCard: true,
      spinWheel: true
    },
    createdAt: new Date().toISOString()
  });

  const [generatedLink, setGeneratedLink] = useState('');
  const [isSaved, setIsSaved] = useState(false);

  const handleSave = () => {
    // In production, sync with Supabase/Firebase here
    const url = `${window.location.origin}/${config.slug}`;
    setGeneratedLink(url);
    setIsSaved(true);
  };

  return (
    <div className="min-h-screen bg-slate-950 text-white p-4 md:p-10 font-sans">
      <div className="max-w-5xl mx-auto">
        {/* Header */}
        <div className="flex flex-col md:flex-row justify-between items-center mb-10 pb-6 border-b border-white/10 gap-4">
          <div>
            <h1 className="text-3xl font-serif font-bold bg-gradient-to-r from-pink-400 to-purple-400 bg-clip-text text-transparent">
              Birthday Gift Builder Studio
            </h1>
            <p className="text-slate-400 text-sm mt-1">Design a magical cinematic experience without writing code.</p>
          </div>
          <motion.button
            whileHover={{ scale: 1.05 }}
            whileTap={{ scale: 0.95 }}
            onClick={handleSave}
            className="px-6 py-3 rounded-2xl bg-gradient-to-r from-pink-500 to-purple-500 font-bold shadow-lg shadow-pink-500/30 flex items-center gap-2"
          >
            <Sparkles className="w-5 h-5" /> Generate Magic Link
          </motion.button>
        </div>

        {/* Builder Sections Grid */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
          {/* General Information */}
          <div className="p-6 rounded-3xl bg-white/5 backdrop-blur-md border border-white/10">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2 text-pink-300">
              <Type className="w-5 h-5" /> Personal Details
            </h2>
            <div className="space-y-4">
              <div>
                <label className="block text-sm text-slate-300 mb-1">Recipient Name</label>
                <input
                  type="text"
                  value={config.recipientName}
                  onChange={(e) => setConfig({ ...config, recipientName: e.target.value })}
                  className="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 focus:outline-none focus:border-pink-500 text-white"
                />
              </div>
              <div>
                <label className="block text-sm text-slate-300 mb-1">Sender Name</label>
                <input
                  type="text"
                  value={config.senderName}
                  onChange={(e) => setConfig({ ...config, senderName: e.target.value })}
                  className="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 focus:outline-none focus:border-pink-500 text-white"
                />
              </div>
              <div>
                <label className="block text-sm text-slate-300 mb-1">Custom URL Slug</label>
                <input
                  type="text"
                  value={config.slug}
                  onChange={(e) => setConfig({ ...config, slug: e.target.value })}
                  className="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 focus:outline-none focus:border-pink-500 text-white"
                />
              </div>
            </div>
          </div>

          {/* Theme & Music Selection */}
          <div className="p-6 rounded-3xl bg-white/5 backdrop-blur-md border border-white/10">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2 text-purple-300">
              <Music className="w-5 h-5" /> Theme & Background Audio
            </h2>
            <div className="space-y-4">
              <div>
                <label className="block text-sm text-slate-300 mb-1">Select Theme</label>
                <select
                  value={config.theme}
                  onChange={(e) => setConfig({ ...config, theme: e.target.value as ThemeMode })}
                  className="w-full px-4 py-3 rounded-xl bg-slate-900 border border-white/20 focus:outline-none focus:border-purple-500 text-white"
                >
                  <option value="romantic">Romantic Rose</option>
                  <option value="elegant">Elegant Gold</option>
                  <option value="cute">Cute Pastel</option>
                  <option value="dark">Midnight Dark</option>
                  <option value="luxury">Luxury Velvet</option>
                  <option value="neon">Cyber Neon</option>
                  <option value="galaxy">Galaxy Dreams</option>
                </select>
              </div>
              <div>
                <label className="block text-sm text-slate-300 mb-1">Background Music URL (MP3)</label>
                <input
                  type="text"
                  value={config.musicUrl}
                  onChange={(e) => setConfig({ ...config, musicUrl: e.target.value })}
                  className="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 focus:outline-none focus:border-purple-500 text-white"
                />
              </div>
            </div>
          </div>

          {/* Personal Letter Editor */}
          <div className="p-6 rounded-3xl bg-white/5 backdrop-blur-md border border-white/10 md:col-span-2">
            <h2 className="text-xl font-semibold mb-4 flex items-center gap-2 text-amber-300">
              <Type className="w-5 h-5" /> Personal Birthday Letter
            </h2>
            <textarea
              rows={5}
              value={config.letterMessage}
              onChange={(e) => setConfig({ ...config, letterMessage: e.target.value })}
              className="w-full px-4 py-3 rounded-xl bg-white/10 border border-white/20 focus:outline-none focus:border-amber-500 text-white font-handwriting text-xl"
              placeholder="Write your heartfelt message here..."
            />
          </div>
        </div>

        {/* Generated Link Modal / Result */}
        {isSaved && (
          <motion.div 
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            className="mt-10 p-8 rounded-3xl bg-gradient-to-r from-pink-500/20 via-purple-500/25 to-indigo-500/20 border border-white/30 backdrop-blur-xl flex flex-col md:flex-row items-center justify-between gap-6"
          >
            <div>
              <div className="flex items-center gap-2 text-emerald-400 font-semibold mb-2">
                <CheckCircle2 className="w-5 h-5" /> Gift Website Created Successfully!
              </div>
              <p className="text-slate-300 text-sm mb-4">Share this unique link with {config.recipientName}:</p>
              <div className="bg-black/40 px-4 py-3 rounded-xl border border-white/10 text-pink-300 font-mono text-sm break-all">
                {generatedLink}
              </div>
            </div>
            <div className="bg-white p-4 rounded-2xl shadow-xl flex flex-col items-center">
              <QRCodeSVG value={generatedLink} size={120} />
              <span className="text-xs text-slate-800 font-medium mt-2">Scan to Open</span>
            </div>
          </motion.div>
        )}
      </div>
    </div>
  );
}

Dynamic Recipient Page (src/app/[slug]/page.tsx)
'default client';

import React, { useState } from 'react';
import { AudioProvider } from '@/context/AudioContext';
import { WelcomeScreen } from '@/components/sections/WelcomeScreen';
import { MemoryGallery } from '@/components/sections/MemoryGallery';
import { BirthdayCake } from '@/components/sections/BirthdayCake';
import { motion } from 'framer-motion';
import confetti from 'canvas-confetti';

export default function BirthdayGiftPage({ params }: { params: { slug: string } }) {
  const [isUnlocked, setIsUnlocked] = useState(false);

  // Mock data fetched by slug
  const giftData = {
    recipientName: "Sarah",
    senderName: "Alex",
    musicUrl: "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3",
    letterMessage: "Happy Birthday to the most wonderful soul! May your year ahead shine brighter than the stars.",
    images: [
      { id: '1', url: 'https://images.unsplash.com/photo-1517841905240-472988babdf9', caption: 'Magical Sunset', date: 'July 2025' },
      { id: '2', url: 'https://images.unsplash.com/photo-1529156069898-49953e39b3ac', caption: 'Best Moments', date: 'December 2025' },
    ]
  };

  const handleOpen = () => {
    setIsUnlocked(true);
    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 }
    });
  };

  return (
    <AudioProvider initialUrl={giftData.musicUrl}>
      <main className="min-h-screen bg-gradient-to-br from-slate-950 via-purple-950 to-slate-900 text-white relative overflow-hidden">
        {/* Welcome Splash Experience */}
        <WelcomeScreen 
          recipientName={giftData.recipientName} 
          senderName={giftData.senderName} 
          onOpen={handleOpen} 
        />

        {/* Main Gift Content */}
        {isUnlocked && (
          <motion.div
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            transition={{ duration: 1 }}
          >
            {/* Header Title Section */}
            <section className="py-20 text-center px-4">
              <span className="text-pink-400 uppercase tracking-widest text-sm font-semibold">Happy Birthday</span>
              <h1 className="text-5xl md:text-7xl font-serif font-extrabold mt-2 mb-4 bg-gradient-to-r from-pink-200 via-white to-purple-300 bg-clip-text text-transparent">
                {giftData.recipientName}
              </h1>
              <p className="text-slate-300 max-w-xl mx-auto text-lg font-handwriting">{giftData.letterMessage}</p>
            </section>

            {/* Memory Gallery */}
            <MemoryGallery images={giftData.images} />

            {/* Birthday Cake Section */}
            <BirthdayCake />
          </motion.div>
        )}
      </main>
    </AudioProvider>
  );
}

