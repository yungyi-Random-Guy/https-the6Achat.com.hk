import React, { useState, useEffect, useRef } from 'react';  
import { initializeApp } from 'firebase/app';  
import { initializeApp } from 'firebase/app';  
import {   
  getAuth,   
  signInWithCustomToken,   
  signInAnonymously,   
  onAuthStateChanged   
} from 'firebase/auth';  
} from 'firebase/auth';  
import {   
import {   
  getFirestore,   
  collection,   
  doc,   
  setDoc,   
  setDoc,   
  addDoc,   
  addDoc,   
  onSnapshot,   
  updateDoc,   
  updateDoc,   
  deleteDoc  
  deleteDoc  
} from 'firebase/firestore';  
import {   
import {   
  MessageSquare,   
  Send,   
  Send,   
  User,   
  User,   
  Users,   
  Users,   
  BookOpen,   
  Plus,   
  Plus,   
  Search,   
  Search,   
  Sparkles,   
  LogOut,   
  Info,   
  Heart,   
  Heart,   
  ThumbsUp,   
  ThumbsUp,   
  Play,   
  ArrowRight,   
  Trash,   
  Trash,   
  Image as ImageIcon,   
  Link as LinkIcon,   
  Code,   
  CheckCircle,  
  Menu,  
  Menu,  
  X  
  X  
} from 'lucide-react';  
  
const firebaseConfig = typeof __firebase_config !== 'undefined'   
const firebaseConfig = typeof __firebase_config !== 'undefined'   
  ? JSON.parse(__firebase_config)   
  ? JSON.parse(__firebase_config)   
  : { apiKey: "", authDomain: "", projectId: "", storageBucket: "", messagingSenderId: "", appId: "" };  
  : { apiKey: "", authDomain: "", projectId: "", storageBucket: "", messagingSenderId: "", appId: "" };  
  
const app = initializeApp(firebaseConfig);  
const app = initializeApp(firebaseConfig);  
const auth = getAuth(app);  
const db = getFirestore(app);  
const db = getFirestore(app);  
const appId = typeof __app_id !== 'undefined' ? __app_id : 'class-6a-hub';  
  
// 產生成員清單 (6A01 - 6A32 + 預設教師與主任)  
const CLASS_MEMBERS = Array.from({ length: 32 }, (_, i) => {  
  const num = String(i + 1).padStart(2, '0');  
  const num = String(i + 1).padStart(2, '0');  
  return `6A${num}`;  
  return `6A${num}`;  
});  
});  
  
const TEACHERS = [  
  { id: 'teacher_hui', name: '許老師', role: '老師' },  
  { id: 'teacher_chu', name: '朱老師', role: '老師' },  
  { id: 'teacher_chu', name: '朱老師', role: '老師' },  
  { id: 'teacher_liao', name: '廖老師', role: '老師' },  
  { id: 'director_tang', name: '唐主任', role: '主任' }  
];  
];  
  
const SECRET_ADMIN = "6A32";  
  
// 粵語與中文常見粗口屏蔽過濾器  
// 粵語與中文常見粗口屏蔽過濾器  
const TOXIC_WORDS = [  
const TOXIC_WORDS = [  
  "屌", "閪", "鳩", "柒", "仆街", "咸濕", "鹹濕", "戇九", "戇𨶙", "戇x", "戇c",   
  "吊你", "操你", "他媽的", "幹你娘", "fuck", "bitch", "shit", "asshole"  
];  
];  
  
const filterBadWords = (text) => {  
  if (!text) return { censoredText: "", isHighlyToxic: false };  
  if (!text) return { censoredText: "", isHighlyToxic: false };  
  let tempText = text;  
  let tempText = text;  
  let matches = 0;  
  TOXIC_WORDS.forEach(word => {  
  TOXIC_WORDS.forEach(word => {  
    const regex = new RegExp(word, "gi");  
    if (regex.test(tempText)) {  
      tempText = tempText.replace(regex, "***");  
      tempText = tempText.replace(regex, "***");  
      matches++;  
      matches++;  
    }  
    }  
  });  
  return { censoredText: tempText, isHighlyToxic: matches >= 4 };  
  return { censoredText: tempText, isHighlyToxic: matches >= 4 };  
};  
};  
  
export default function App() {  
export default function App() {  
  const [user, setUser] = useState(null);  
  const [selectedMember, setSelectedMember] = useState("");  
  const [isJoined, setIsJoined] = useState(false);  
  const [isJoined, setIsJoined] = useState(false);  
  const [loading, setLoading] = useState(false);  
  const [loading, setLoading] = useState(false);  
  
  // 系統導航 (chat: 聊天, instagram: 動態, voting: 投票, kahoot: 問答, admin: 隱藏維護)  
  // 系統導航 (chat: 聊天, instagram: 動態, voting: 投票, kahoot: 問答, admin: 隱藏維護)  
  const [currentTab, setCurrentTab] = useState("chat");  
  
  // 聊天室 & 私人密聊狀態  
  // 聊天室 & 私人密聊狀態  
  const [rooms, setRooms] = useState([]);  
  const [activeRoomId, setActiveRoomId] = useState("global");  
  const [messages, setMessages] = useState([]);  
  const [messages, setMessages] = useState([]);  
  const [newMessage, setNewMessage] = useState("");  
  const [chatImageBase64, setChatImageBase64] = useState("");  
  const [chatImageBase64, setChatImageBase64] = useState("");  
  const [chatLinkUrl, setChatLinkUrl] = useState("");  
  const [chatLinkUrl, setChatLinkUrl] = useState("");  
  const [chatLinkTitle, setChatLinkTitle] = useState("");  
  const [htmlCode, setHtmlCode] = useState("");  
    
    
  // Modals 與選單  
  const [showHtmlModal, setShowHtmlModal] = useState(false);  
  const [showHtmlModal, setShowHtmlModal] = useState(false);  
  const [showLinkModal, setShowLinkModal] = useState(false);  
  const [showLinkModal, setShowLinkModal] = useState(false);  
  const [showCreateRoomModal, setShowCreateRoomModal] = useState(false);  
  const [newRoomParticipants, setNewRoomParticipants] = useState([]);  
  const [newRoomParticipants, setNewRoomParticipants] = useState([]);  
  const [newGroupName, setNewGroupName] = useState("");  
  const [activeHtmlPayload, setActiveHtmlPayload] = useState(null);  
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);  
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);  
  
  // Open Mind Pro (維基百科 AI) 的歷史條目緩存與本地知識庫  
  // Open Mind Pro (維基百科 AI) 的歷史條目緩存與本地知識庫  
  const [lastWikiTopic, setLastWikiTopic] = useState("");  
  const [lastWikiTopic, setLastWikiTopic] = useState("");  
  const [lastWikiContent, setLastWikiContent] = useState("");  
  const [lastWikiContent, setLastWikiContent] = useState("");  
  const [infoCentre, setInfoCentre] = useState({});  
  
  // Instagram 狀態  
  const [posts, setPosts] = useState([]);  
  const [posts, setPosts] = useState([]);  
  const [newPostText, setNewPostText] = useState("");  
  const [newPostText, setNewPostText] = useState("");  
  const [newPostImageBase64, setNewPostImageBase64] = useState("");  
  const [newPostImageBase64, setNewPostImageBase64] = useState("");  
  const [showCreatePost, setShowCreatePost] = useState(false);  
  const [showCreatePost, setShowCreatePost] = useState(false);  
  
  // 投票公投狀態  
  // 投票公投狀態  
  const [polls, setPolls] = useState([]);  
  const [newProposalTitle, setNewProposalTitle] = useState("");  
  const [newProposalTitle, setNewProposalTitle] = useState("");  
  const [newProposalDesc, setNewProposalDesc] = useState("");  
  const [showCreatePoll, setShowCreatePoll] = useState(false);  
  const [showCreatePoll, setShowCreatePoll] = useState(false);  
  
  // Kahoot! 狀態  
  // Kahoot! 狀態  
  const [quizzes, setQuizzes] = useState([]);  
  const [quizScores, setQuizScores] = useState([]);  
  const [quizScores, setQuizScores] = useState([]);  
  const [activeQuiz, setActiveQuiz] = useState(null);  
  const [activeQuiz, setActiveQuiz] = useState(null);  
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);  
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);  
  const [quizScore, setQuizScore] = useState(0);  
  const [quizTimer, setQuizTimer] = useState(15);  
  const [quizTimer, setQuizTimer] = useState(15);  
  const [quizFinished, setQuizFinished] = useState(false);  
  const [quizFinished, setQuizFinished] = useState(false);  
  const [selectedAnswerIndex, setSelectedAnswerIndex] = useState(null);  
  const [selectedAnswerIndex, setSelectedAnswerIndex] = useState(null);  
  const [showCreateQuiz, setShowCreateQuiz] = useState(false);  
  const [showCreateQuiz, setShowCreateQuiz] = useState(false);  
  
  // 建立 Kahoot! 題目  
  const [newQuizTitle, setNewQuizTitle] = useState("");  
  const [newQuizTitle, setNewQuizTitle] = useState("");  
  const [newQuizQuestions, setNewQuizQuestions] = useState([  
    { question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }  
    { question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }  
  ]);  
  
  // 禁言/成員權限  
  // 禁言/成員權限  
  const [memberStatus, setMemberStatus] = useState({});  
  const [memberStatus, setMemberStatus] = useState({});  
  
  const chatEndRef = useRef(null);  
  
  useEffect(() => {  
  useEffect(() => {  
    const initAuth = async () => {  
      try {  
      try {  
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {  
          await signInWithCustomToken(auth, __initial_auth_token);  
          await signInWithCustomToken(auth, __initial_auth_token);  
        } else {  
          await signInAnonymously(auth);  
        }  
        }  
      } catch (err) {  
      } catch (err) {  
        console.error("Auth initialization failed:", err);  
        console.error("Auth initialization failed:", err);  
      }  
      }  
    };  
    };  
    initAuth();  
    initAuth();  
  
    const unsubscribe = onAuthStateChanged(auth, (authUser) => {  
    const unsubscribe = onAuthStateChanged(auth, (authUser) => {  
      if (authUser) {  
      if (authUser) {  
        const savedMember = localStorage.getItem("6a_active_member");  
        const savedMember = localStorage.getItem("6a_active_member");  
        if (savedMember) {  
        if (savedMember) {  
          setSelectedMember(savedMember);  
          setIsJoined(true);  
          setIsJoined(true);  
          setUser({  
          setUser({  
            uid: authUser.uid,  
            memberId: savedMember,  
            memberId: savedMember,  
            isAdmin: savedMember === SECRET_ADMIN  
          });  
        }  
        }  
      } else {  
      } else {  
        setUser(null);  
        setUser(null);  
      }  
      }  
    });  
    });  
  
    return () => unsubscribe();  
  }, []);  
  }, []);  
  
  useEffect(() => {  
    if (!user || !isJoined) return;  
    if (!user || !isJoined) return;  
  
    // 1. 監聽聊天室名單  
    const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
    const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
    const unsubscribeRooms = onSnapshot(roomsCollection, (snapshot) => {  
    const unsubscribeRooms = onSnapshot(roomsCollection, (snapshot) => {  
      const roomList = [];  
      const roomList = [];  
      snapshot.forEach((doc) => {  
        roomList.push({ id: doc.id, ...doc.data() });  
      });  
      setRooms(roomList);  
    });  
    });  
  
    // 2. 監聽所有聊天訊息  
    // 2. 監聽所有聊天訊息  
    const chatCollection = collection(db, 'artifacts', appId, 'public', 'data', 'chats');  
    const chatCollection = collection(db, 'artifacts', appId, 'public', 'data', 'chats');  
    const unsubscribeChats = onSnapshot(chatCollection, (snapshot) => {  
    const unsubscribeChats = onSnapshot(chatCollection, (snapshot) => {  
      const msgList = [];  
      snapshot.forEach((doc) => {  
        msgList.push({ id: doc.id, ...doc.data() });  
      });  
      });  
      msgList.sort((a, b) => (a.createdAt || 0) - (b.createdAt || 0));  
      msgList.sort((a, b) => (a.createdAt || 0) - (b.createdAt || 0));  
      setMessages(msgList);  
    });  
    });  
  
    // 3. 監聽 Instagram 動態貼文  
    const postsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'posts');  
    const unsubscribePosts = onSnapshot(postsCollection, (snapshot) => {  
    const unsubscribePosts = onSnapshot(postsCollection, (snapshot) => {  
      const postList = [];  
      const postList = [];  
      snapshot.forEach((doc) => {  
        postList.push({ id: doc.id, ...doc.data() });  
        postList.push({ id: doc.id, ...doc.data() });  
      });  
      postList.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));  
      setPosts(postList);  
    });  
  
    // 4. 監聽建議功能投票  
    // 4. 監聽建議功能投票  
    const pollsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'polls');  
    const unsubscribePolls = onSnapshot(pollsCollection, (snapshot) => {  
    const unsubscribePolls = onSnapshot(pollsCollection, (snapshot) => {  
      const pollList = [];  
      const pollList = [];  
      snapshot.forEach((doc) => {  
      snapshot.forEach((doc) => {  
        pollList.push({ id: doc.id, ...doc.data() });  
        pollList.push({ id: doc.id, ...doc.data() });  
      });  
      });  
      pollList.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));  
      setPolls(pollList);  
      setPolls(pollList);  
    });  
  
    // 5. 監聽 Kahoot! 題庫  
    // 5. 監聽 Kahoot! 題庫  
    const quizzesCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quizzes');  
    const quizzesCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quizzes');  
    const unsubscribeQuizzes = onSnapshot(quizzesCollection, (snapshot) => {  
      const quizList = [];  
      snapshot.forEach((doc) => {  
      snapshot.forEach((doc) => {  
        quizList.push({ id: doc.id, ...doc.data() });  
        quizList.push({ id: doc.id, ...doc.data() });  
      });  
      setQuizzes(quizList);  
      setQuizzes(quizList);  
    });  
    });  
  
    // 6. 監聽 Kahoot! 排行得分  
    const scoresCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quiz_scores');  
    const unsubscribeScores = onSnapshot(scoresCollection, (snapshot) => {  
      const scoreList = [];  
      snapshot.forEach((doc) => {  
      snapshot.forEach((doc) => {  
        scoreList.push({ id: doc.id, ...doc.data() });  
        scoreList.push({ id: doc.id, ...doc.data() });  
      });  
      });  
      scoreList.sort((a, b) => (b.score || 0) - (a.score || 0));  
      scoreList.sort((a, b) => (b.score || 0) - (a.score || 0));  
      setQuizScores(scoreList);  
      setQuizScores(scoreList);  
    });  
    });  
  
    // 7. 監聽本地知識庫 (Information Centre)  
    // 7. 監聽本地知識庫 (Information Centre)  
    const infoCollection = collection(db, 'artifacts', appId, 'public', 'data', 'information_centre');  
    const infoCollection = collection(db, 'artifacts', appId, 'public', 'data', 'information_centre');  
    const unsubscribeInfo = onSnapshot(infoCollection, (snapshot) => {  
    const unsubscribeInfo = onSnapshot(infoCollection, (snapshot) => {  
      const infoMap = {};  
      const infoMap = {};  
      snapshot.forEach((doc) => {  
      snapshot.forEach((doc) => {  
        const d = doc.data();  
        const d = doc.data();  
        infoMap[d.keyword.toLowerCase()] = {  
          content: d.content,  
          content: d.content,  
          author: d.author  
          author: d.author  
        };  
      });  
      });  
      setInfoCentre(infoMap);  
    });  
  
    // 8. 監聽成員禁言狀態  
    const statusCollection = collection(db, 'artifacts', appId, 'public', 'data', 'status');  
    const statusCollection = collection(db, 'artifacts', appId, 'public', 'data', 'status');  
    const unsubscribeStatus = onSnapshot(statusCollection, (snapshot) => {  
      const statusMap = {};  
      const statusMap = {};  
      snapshot.forEach((doc) => {  
      snapshot.forEach((doc) => {  
        statusMap[doc.id] = doc.data();  
      });  
      });  
      setMemberStatus(statusMap);  
    });  
  
    return () => {  
    return () => {  
      unsubscribeRooms();  
      unsubscribeRooms();  
      unsubscribeChats();  
      unsubscribePosts();  
      unsubscribePolls();  
      unsubscribeQuizzes();  
      unsubscribeQuizzes();  
      unsubscribeScores();  
      unsubscribeInfo();  
      unsubscribeStatus();  
      unsubscribeStatus();  
    };  
    };  
  }, [user, isJoined]);  
  }, [user, isJoined]);  
  
  // 聊天室自動置底  
  // 聊天室自動置底  
  useEffect(() => {  
  useEffect(() => {  
    chatEndRef.current?.scrollIntoView({ behavior: 'smooth' });  
    chatEndRef.current?.scrollIntoView({ behavior: 'smooth' });  
  }, [messages, activeRoomId, currentTab]);  
  
  // Kahoot! 限時倒數計時  
  useEffect(() => {  
  useEffect(() => {  
    if (activeQuiz && !quizFinished) {  
    if (activeQuiz && !quizFinished) {  
      const currentQ = activeQuiz.questions[currentQuestionIndex];  
      const currentQ = activeQuiz.questions[currentQuestionIndex];  
      if (!currentQ) return;  
        
        
      const timer = setInterval(() => {  
      const timer = setInterval(() => {  
        setQuizTimer(prev => {  
        setQuizTimer(prev => {  
          if (prev <= 1) {  
            clearInterval(timer);  
            clearInterval(timer);  
            handleNextQuestion(null);  
            return currentQ.timeLimit || 15;  
          }  
          return prev - 1;  
        });  
        });  
      }, 1000);  
      }, 1000);  
  
      return () => clearInterval(timer);  
      return () => clearInterval(timer);  
    }  
    }  
  }, [activeQuiz, currentQuestionIndex, quizFinished]);  
  }, [activeQuiz, currentQuestionIndex, quizFinished]);  
  
  const isBanned = () => {  
  const isBanned = () => {  
    if (!user) return false;  
    if (!user) return false;  
    return memberStatus[user.memberId]?.isBanned || false;  
  };  
  };  
  
  const handleLocalImageUpload = (e, callback) => {  
    const file = e.target.files[0];  
    if (!file) return;  
    if (!file) return;  
  
    const reader = new FileReader();  
    reader.readAsDataURL(file);  
    reader.onload = (event) => {  
      const img = new Image();  
      const img = new Image();  
      img.src = event.target.result;  
      img.src = event.target.result;  
      img.onload = () => {  
      img.onload = () => {  
        const canvas = document.createElement('canvas');  
        const canvas = document.createElement('canvas');  
        const MAX_WIDTH = 400;  
        const MAX_HEIGHT = 400;  
        const MAX_HEIGHT = 400;  
        let width = img.width;  
        let height = img.height;  
        let height = img.height;  
  
        if (width > height) {  
        if (width > height) {  
          if (width > MAX_WIDTH) {  
          if (width > MAX_WIDTH) {  
            height *= MAX_WIDTH / width;  
            width = MAX_WIDTH;  
          }  
          }  
        } else {  
        } else {  
          if (height > MAX_HEIGHT) {  
          if (height > MAX_HEIGHT) {  
            width *= MAX_HEIGHT / height;  
            height = MAX_HEIGHT;  
          }  
          }  
        }  
        canvas.width = width;  
        canvas.width = width;  
        canvas.height = height;  
        const ctx = canvas.getContext('2d');  
        ctx.drawImage(img, 0, 0, width, height);  
        ctx.drawImage(img, 0, 0, width, height);  
        const compressedBase64 = canvas.toDataURL('image/jpeg', 0.6);  
        callback(compressedBase64);  
        callback(compressedBase64);  
      };  
      };  
    };  
    };  
  };  
  
  const handleLogin = (e) => {  
    e.preventDefault();  
    if (!selectedMember) return;  
    if (!selectedMember) return;  
    setLoading(true);  
      
      
    localStorage.setItem("6a_active_member", selectedMember);  
    setIsJoined(true);  
    setUser({  
      uid: auth.currentUser?.uid || "anon-6a",  
      uid: auth.currentUser?.uid || "anon-6a",  
      memberId: selectedMember,  
      isAdmin: selectedMember === SECRET_ADMIN  
      isAdmin: selectedMember === SECRET_ADMIN  
    });  
    });  
    setLoading(false);  
  };  
  };  
  
  const handleLogout = () => {  
    localStorage.removeItem("6a_active_member");  
    setIsJoined(false);  
    setUser(null);  
  };  
  };  
  
  const queryWiki = async (topic) => {  
  const queryWiki = async (topic) => {  
    try {  
    try {  
      const searchUrl = `https://zh.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(topic)}&format=json&origin=*`;  
      const searchRes = await fetch(searchUrl);  
      const searchRes = await fetch(searchUrl);  
      const searchData = await searchRes.json();  
      const searchData = await searchRes.json();  
        
        
      if (searchData.query?.search?.length > 0) {  
        const bestMatch = searchData.query.search[0];  
        const pageTitle = bestMatch.title;  
  
        const extractUrl = `https://zh.wikipedia.org/w/api.php?action=query&prop=extracts&exintro&explaintext&titles=${encodeURIComponent(pageTitle)}&format=json&origin=*`;  
        const extractRes = await fetch(extractUrl);  
        const extractRes = await fetch(extractUrl);  
        const extractData = await extractRes.json();  
        const extractData = await extractRes.json();  
          
        const pages = extractData.query?.pages;  
        const pageId = Object.keys(pages)[0];  
        const pageId = Object.keys(pages)[0];  
        const extractText = pages[pageId].extract || "";  
  
        setLastWikiTopic(pageTitle);  
        setLastWikiContent(extractText);  
        setLastWikiContent(extractText);  
  
        return {  
        return {  
          title: pageTitle,  
          title: pageTitle,  
          summary: extractText.substring(0, 150) + (extractText.length > 150 ? "..." : ""),  
          summary: extractText.substring(0, 150) + (extractText.length > 150 ? "..." : ""),  
          full: extractText,  
          full: extractText,  
          url: `https://zh.wikipedia.org/wiki/${encodeURIComponent(pageTitle)}`  
        };  
      }  
      }  
      return null;  
      return null;  
    } catch (e) {  
      console.error("Wikipedia API fetch error:", e);  
      return null;  
      return null;  
    }  
  };  
  };  
  
  const speakVoice = (text) => {  
    if ('speechSynthesis' in window) {  
    if ('speechSynthesis' in window) {  
      window.speechSynthesis.cancel();  
      const utterance = new SpeechSynthesisUtterance(text.substring(0, 160));  
      utterance.lang = "zh-HK";  
      utterance.rate = 1.0;  
      utterance.rate = 1.0;  
      window.speechSynthesis.speak(utterance);  
    }  
    }  
  };  
  };  
  
  const getNaturalAIResponse = async (inputStr) => {  
    const textLower = inputStr.trim().toLowerCase();  
    const textLower = inputStr.trim().toLowerCase();  
      
      
    // 1. 學習指令: omp learn [keyword]: [content]  
    const learnRegex = /^(?:omp|openmind|open mind)\s+learn\s+([^:]+):(.*)/i;  
    const learnRegex = /^(?:omp|openmind|open mind)\s+learn\s+([^:]+):(.*)/i;  
    if (learnRegex.test(textLower)) {  
    if (learnRegex.test(textLower)) {  
      const match = inputStr.match(learnRegex);  
      const match = inputStr.match(learnRegex);  
      if (match) {  
      if (match) {  
        const keyword = match[1].trim();  
        const keyword = match[1].trim();  
        const content = match[2].trim();  
        const content = match[2].trim();  
        if (keyword && content) {  
        if (keyword && content) {  
          const infoDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'information_centre', encodeURIComponent(keyword.toLowerCase()));  
          const infoDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'information_centre', encodeURIComponent(keyword.toLowerCase()));  
          await setDoc(infoDocRef, {  
          await setDoc(infoDocRef, {  
            keyword,  
            content,  
            content,  
            author: user.memberId,  
            author: user.memberId,  
            timestamp: Date.now()  
          });  
          });  
  
          const responses = [  
          const responses = [  
            `嘩！真係唔該晒你呀，等我快啲記底先。📝 我已經將「${keyword}」儲存到資訊中心啦！下次有人問我，我就會大大聲話俾佢地聽：『${content}』！`,  
            `收到！「${keyword}」原來係咁解，我又學到新嘢啦！等我將佢記落我個超級大腦度。多謝你教我呀！🧠`,  
            `好嘢！我又聰明左啦。關於「${keyword}」嘅解釋已經存入左資訊中心。多謝你呀，你真係好幫得手！✨`  
          ];  
          ];  
          const randomReply = responses[Math.floor(Math.random() * responses.length)];  
          speakVoice(`多謝你教我呀！我已經記底咗關於${keyword}嘅知識啦。`);  
          speakVoice(`多謝你教我呀！我已經記底咗關於${keyword}嘅知識啦。`);  
          return randomReply;  
        }  
        }  
      }  
    }  
  
    // 2. 請求更多/MOR  
    if (textLower === "更多" || textLower === "mor" || textLower === "more") {  
    if (textLower === "更多" || textLower === "mor" || textLower === "more") {  
      if (lastWikiTopic && lastWikiContent) {  
        speakVoice(`好啊，等我同你講多啲關於 ${lastWikiTopic} 嘅內容啦。`);  
        return `好啊！等我將知道關於《${lastWikiTopic}》嘅全部細節都話晒你知，你想睇多啲對不對？👇\n\n${lastWikiContent}\n\n希望呢啲資料幫到你啦！🥰`;  
        return `好啊！等我將知道關於《${lastWikiTopic}》嘅全部細節都話晒你知，你想睇多啲對不對？👇\n\n${lastWikiContent}\n\n希望呢啲資料幫到你啦！🥰`;  
      } else {  
        return "咦？你係咪想聽多啲？不過我依家個腦袋空空如也，你仲未同我開始任何話題啵。試下同我講「omp [你想知嘅野]」先啦！🔍";  
        return "咦？你係咪想聽多啲？不過我依家個腦袋空空如也，你仲未同我開始任何話題啵。試下同我講「omp [你想知嘅野]」先啦！🔍";  
      }  
      }  
    }  
  
    // 3. 常規問候  
    // 3. 常規問候  
    const greetings = ["你好", "嗨", "hello", "hi", "喂", "在嗎", "早晨", "早安"];  
    const greetings = ["你好", "嗨", "hello", "hi", "喂", "在嗎", "早晨", "早安"];  
    if (greetings.some(g => textLower.includes(g))) {  
      const responses = [  
        `嗨！你好呀！今日有咩好玩嘅事分享？定係你想查啲咩？話我知啦！😊`,  
        `嗨！你好呀！今日有咩好玩嘅事分享？定係你想查啲咩？話我知啦！😊`,  
        `喂！今日過得點呀？我係 Open Mind Pro，今日想同我傾啲咩？`,  
        `Hello！見到你真係高興！今日等我隨時為你提供維基百科同班級資訊！✨`  
      ];  
      ];  
      return responses[Math.floor(Math.random() * responses.length)];  
      return responses[Math.floor(Math.random() * responses.length)];  
    }  
    }  
  
    // 4. 百科或資訊中心查詢  
    // 4. 百科或資訊中心查詢  
    let queryTerm = inputStr.replace(/^(omp|openmind|open mind)\s+/i, "").trim();  
    let queryTerm = inputStr.replace(/^(omp|openmind|open mind)\s+/i, "").trim();  
    const isAiPrivateRoom = activeRoomId.startsWith("private-omp-");  
    const isAiPrivateRoom = activeRoomId.startsWith("private-omp-");  
      
    if (!queryTerm && isAiPrivateRoom) {  
      queryTerm = inputStr.trim();  
    }  
    }  
  
    if (queryTerm) {  
      // 先查維基百科  
      // 先查維基百科  
      const wiki = await queryWiki(queryTerm);  
      const wiki = await queryWiki(queryTerm);  
      if (wiki) {  
      if (wiki) {  
        speakVoice(wiki.summary);  
        speakVoice(wiki.summary);  
        return `等我幫你上維基百科查吓先…… 🔍 搵到啦！原來【${wiki.title}】係咁樣嘅：\n\n「${wiki.summary}」\n\n如果你想睇埋其餘部分，可以同我講「更多」或者「MOR」㗎！\n🔗 傳送門：${wiki.url}`;  
        return `等我幫你上維基百科查吓先…… 🔍 搵到啦！原來【${wiki.title}】係咁樣嘅：\n\n「${wiki.summary}」\n\n如果你想睇埋其餘部分，可以同我講「更多」或者「MOR」㗎！\n🔗 傳送門：${wiki.url}`;  
      }  
  
      // 查本地資訊中心  
      // 查本地資訊中心  
      const local = infoCentre[queryTerm.toLowerCase()];  
      const local = infoCentre[queryTerm.toLowerCase()];  
      if (local) {  
        speakVoice(local.content);  
        speakVoice(local.content);  
        return `咦！維基百科就查唔到，不過好在之前有同學喺「資訊中心」教過我！💡\n\n原來【${queryTerm}】係指：\n「${local.content}」\n\n呢個係由 ${local.author} 同學話我知嘅！`;  
        return `咦！維基百科就查唔到，不過好在之前有同學喺「資訊中心」教過我！💡\n\n原來【${queryTerm}】係指：\n「${local.content}」\n\n呢個係由 ${local.author} 同學話我知嘅！`;  
      }  
      }  
  
      // 兩邊找不到，請求使用者教導  
      // 兩邊找不到，請求使用者教導  
      const questionResponses = [  
        `哎呀，我查過維基百科同埋我個小腦袋，都唔知咩係「${queryTerm}」喎…… 😢\n\n你可唔可以做我老師教吓我呀？\n只要打：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n我就會一世記住你教我嘅嘢㗎啦！`,  
        `唔好意思呀，我喺維基同埋資訊中心都搵唔到「${queryTerm}」嘅資料。🧐\n\n不如你教吓我啦？你可以用以下格式教我：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n拜託你啦！`,  
        `咦？「${queryTerm}」係啲咩黎㗎？聽落好深奧，連維基都無寫！\n\n你可唔可以喺度解釋俾我聽？請打：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n我等緊你教我㗎！🎒`  
        `咦？「${queryTerm}」係啲咩黎㗎？聽落好深奧，連維基都無寫！\n\n你可唔可以喺度解釋俾我聽？請打：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n我等緊你教我㗎！🎒`  
      ];  
      ];  
      speakVoice(`我真係唔識得咩係${queryTerm}呀，你可以教我嗎？`);  
      return questionResponses[Math.floor(Math.random() * questionResponses.length)];  
    }  
    }  
  
    return "嗯…… 我好似聽唔係好明，你可以試下輸入「omp [你想查嘅主題]」或者同我隨便傾兩句呀！";  
  };  
  };  
  
  const handleSendMessage = async (e) => {  
    e.preventDefault();  
    e.preventDefault();  
    if (isBanned()) {  
      alert("傳送失敗：系統通訊異常，請稍後再試。");  
      alert("傳送失敗：系統通訊異常，請稍後再試。");  
      return;  
    }  
  
    const { censoredText, isHighlyToxic } = filterBadWords(newMessage);  
    if (isHighlyToxic) {  
    if (isHighlyToxic) {  
      alert("⚠️ 請注意班規及發言禮儀！留言包含不當詞彙。");  
      return;  
      return;  
    }  
    }  
  
    if (!censoredText.trim() && !chatImageBase64 && !htmlCode.trim() && !chatLinkUrl) return;  
  
    try {  
      const chatCollection = collection(db, 'artifacts', appId, 'public', 'data', 'chats');  
      const chatCollection = collection(db, 'artifacts', appId, 'public', 'data', 'chats');  
      await addDoc(chatCollection, {  
      await addDoc(chatCollection, {  
        sender: user.memberId,  
        sender: user.memberId,  
        roomId: activeRoomId,  
        text: censoredText,  
        text: censoredText,  
        image: chatImageBase64 || null,  
        image: chatImageBase64 || null,  
        html: htmlCode || null,  
        link: chatLinkUrl ? { url: chatLinkUrl, title: chatLinkTitle || "分享連結" } : null,  
        createdAt: Date.now()  
        createdAt: Date.now()  
      });  
      });  
  
      const currentMsgText = censoredText;  
  
      setNewMessage("");  
      setNewMessage("");  
      setChatImageBase64("");  
      setHtmlCode("");  
      setHtmlCode("");  
      setChatLinkUrl("");  
      setChatLinkUrl("");  
      setChatLinkTitle("");  
      setShowHtmlModal(false);  
      setShowHtmlModal(false);  
      setShowLinkModal(false);  
      setShowLinkModal(false);  
  
      // 觸發 AI 回應  
      // 觸發 AI 回應  
      const isAiRoom = activeRoomId.startsWith("private-omp-");  
      const isAiRoom = activeRoomId.startsWith("private-omp-");  
      const isOmpTrigger = /^(omp|openmind|open mind)\s+/i.test(currentMsgText.toLowerCase()) ||   
      const isOmpTrigger = /^(omp|openmind|open mind)\s+/i.test(currentMsgText.toLowerCase()) ||   
                           currentMsgText.toLowerCase() === "更多" ||   
                           currentMsgText.toLowerCase() === "更多" ||   
                           currentMsgText.toLowerCase() === "mor" ||  
                           currentMsgText.toLowerCase() === "mor" ||  
                           currentMsgText.toLowerCase() === "more";  
  
      if (isAiRoom || isOmpTrigger) {  
      if (isAiRoom || isOmpTrigger) {  
        setTimeout(async () => {  
          const aiResponse = await getNaturalAIResponse(currentMsgText);  
          await addDoc(chatCollection, {  
            sender: "Open Mind Pro 🤖",  
            roomId: activeRoomId,  
            roomId: activeRoomId,  
            text: aiResponse,  
            text: aiResponse,  
            createdAt: Date.now()  
            createdAt: Date.now()  
          });  
          });  
        }, 1000);  
        }, 1000);  
      }  
      }  
  
    } catch (err) {  
    } catch (err) {  
      console.error("發送失敗:", err);  
    }  
  };  
  
  const handleCreateRoom = async (e) => {  
  const handleCreateRoom = async (e) => {  
    e.preventDefault();  
    if (newRoomParticipants.length === 0) return;  
    if (newRoomParticipants.length === 0) return;  
  
    const allParticipants = Array.from(new Set([user.memberId, ...newRoomParticipants])).sort();  
    const allParticipants = Array.from(new Set([user.memberId, ...newRoomParticipants])).sort();  
      
      
    // 檢查是否已存在該對話  
    if (allParticipants.length === 2) {  
      const existing = rooms.find(r =>   
        r.participants?.length === 2 &&   
        r.participants?.length === 2 &&   
        r.participants.includes(allParticipants[0]) &&   
        r.participants.includes(allParticipants[1])  
        r.participants.includes(allParticipants[1])  
      );  
      );  
      if (existing) {  
      if (existing) {  
        setActiveRoomId(existing.id);  
        setShowCreateRoomModal(false);  
        setNewRoomParticipants([]);  
        setNewGroupName("");  
        setNewGroupName("");  
        return;  
      }  
    }  
  
    try {  
    try {  
      const name = newGroupName.trim() || allParticipants.filter(p => p !== user.memberId).join(", ");  
      const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
      const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
      const docRef = await addDoc(roomsCollection, {  
      const docRef = await addDoc(roomsCollection, {  
        name: name,  
        name: name,  
        participants: allParticipants,  
        participants: allParticipants,  
        isGroup: allParticipants.length > 2,  
        isGroup: allParticipants.length > 2,  
        createdBy: user.memberId,  
        createdAt: Date.now()  
        createdAt: Date.now()  
      });  
      });  
  
      setActiveRoomId(docRef.id);  
      setActiveRoomId(docRef.id);  
      setShowCreateRoomModal(false);  
      setShowCreateRoomModal(false);  
      setNewRoomParticipants([]);  
      setNewRoomParticipants([]);  
      setNewGroupName("");  
      setNewGroupName("");  
    } catch (err) {  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleStartTeacherChat = async (teacher) => {  
    const allParticipants = [user.memberId, teacher.name].sort();  
    const existing = rooms.find(r =>   
      r.participants?.length === 2 &&   
      r.participants?.length === 2 &&   
      r.participants.includes(allParticipants[0]) &&   
      r.participants.includes(allParticipants[0]) &&   
      r.participants.includes(allParticipants[1])  
      r.participants.includes(allParticipants[1])  
    );  
  
    if (existing) {  
      setActiveRoomId(existing.id);  
      setCurrentTab("chat");  
      return;  
    }  
  
    try {  
      const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
      const roomsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');  
      const docRef = await addDoc(roomsCollection, {  
      const docRef = await addDoc(roomsCollection, {  
        name: `${teacher.name} (${teacher.role})`,  
        participants: allParticipants,  
        isGroup: false,  
        createdBy: user.memberId,  
        createdBy: user.memberId,  
        createdAt: Date.now()  
        createdAt: Date.now()  
      });  
      });  
  
      setActiveRoomId(docRef.id);  
      setCurrentTab("chat");  
      setCurrentTab("chat");  
    } catch (err) {  
    } catch (err) {  
      console.error(err);  
    }  
  };  
  };  
  
  const handleCreatePost = async (e) => {  
    e.preventDefault();  
    e.preventDefault();  
    if (isBanned()) {  
    if (isBanned()) {  
      alert("發表失敗：系統通訊異常。");  
      return;  
      return;  
    }  
    }  
    if (!newPostText.trim() && !newPostImageBase64) return;  
  
    const { censoredText, isHighlyToxic } = filterBadWords(newPostText);  
    if (isHighlyToxic) {  
    if (isHighlyToxic) {  
      alert("⚠️ 請注意貼文內容禮儀！包含敏感詞。");  
      alert("⚠️ 請注意貼文內容禮儀！包含敏感詞。");  
      return;  
    }  
    }  
  
    try {  
    try {  
      const postsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'posts');  
      const postsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'posts');  
      await addDoc(postsCollection, {  
      await addDoc(postsCollection, {  
        author: user.memberId,  
        author: user.memberId,  
        text: censoredText,  
        text: censoredText,  
        image: newPostImageBase64 || "https://images.unsplash.com/photo-1517483000871-1dbf64a6e1c6?w=400&auto=format&fit=crop",  
        likes: [],  
        createdAt: Date.now()  
        createdAt: Date.now()  
      });  
      });  
  
      setNewPostText("");  
      setNewPostText("");  
      setNewPostImageBase64("");  
      setNewPostImageBase64("");  
      setShowCreatePost(false);  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleLikePost = async (postId, currentLikes = []) => {  
  const handleLikePost = async (postId, currentLikes = []) => {  
    const isLiked = currentLikes.includes(user.memberId);  
    const isLiked = currentLikes.includes(user.memberId);  
    let updatedLikes = isLiked   
      ? currentLikes.filter(id => id !== user.memberId)  
      : [...currentLikes, user.memberId];  
      : [...currentLikes, user.memberId];  
  
    try {  
    try {  
      const postDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'posts', postId);  
      const postDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'posts', postId);  
      await updateDoc(postDocRef, { likes: updatedLikes });  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleCreateProposal = async (e) => {  
  const handleCreateProposal = async (e) => {  
    e.preventDefault();  
    if (!newProposalTitle.trim()) return;  
  
    try {  
    try {  
      const pollsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'polls');  
      const pollsCollection = collection(db, 'artifacts', appId, 'public', 'data', 'polls');  
      await addDoc(pollsCollection, {  
        proposer: user.memberId,  
        proposer: user.memberId,  
        title: newProposalTitle,  
        title: newProposalTitle,  
        description: newProposalDesc,  
        description: newProposalDesc,  
        votes: [],  
        votes: [],  
        createdAt: Date.now()  
      });  
      });  
  
      setNewProposalTitle("");  
      setNewProposalTitle("");  
      setNewProposalDesc("");  
      setShowCreatePoll(false);  
      setShowCreatePoll(false);  
    } catch (err) {  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleVotePoll = async (pollId, currentVotes = []) => {  
  const handleVotePoll = async (pollId, currentVotes = []) => {  
    const hasVoted = currentVotes.includes(user.memberId);  
    let updatedVotes = hasVoted   
      ? currentVotes.filter(id => id !== user.memberId)  
      : [...currentVotes, user.memberId];  
  
    try {  
    try {  
      const pollDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'polls', pollId);  
      await updateDoc(pollDocRef, { votes: updatedVotes });  
      await updateDoc(pollDocRef, { votes: updatedVotes });  
    } catch (err) {  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleCreateQuiz = async (e) => {  
    e.preventDefault();  
    e.preventDefault();  
    if (!newQuizTitle.trim() || newQuizQuestions.some(q => !q.question.trim())) return;  
  
    try {  
    try {  
      const quizzesCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quizzes');  
      const quizzesCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quizzes');  
      await addDoc(quizzesCollection, {  
      await addDoc(quizzesCollection, {  
        creator: user.memberId,  
        title: newQuizTitle,  
        questions: newQuizQuestions,  
        createdAt: Date.now()  
        createdAt: Date.now()  
      });  
  
      setNewQuizTitle("");  
      setNewQuizTitle("");  
      setNewQuizQuestions([{ question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }]);  
      setNewQuizQuestions([{ question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }]);  
      setShowCreateQuiz(false);  
      setShowCreateQuiz(false);  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const handleStartQuiz = (quiz) => {  
  const handleStartQuiz = (quiz) => {  
    setActiveQuiz(quiz);  
    setActiveQuiz(quiz);  
    setCurrentQuestionIndex(0);  
    setCurrentQuestionIndex(0);  
    setQuizScore(0);  
    setQuizScore(0);  
    setQuizFinished(false);  
    setQuizFinished(false);  
    setSelectedAnswerIndex(null);  
    setSelectedAnswerIndex(null);  
    setQuizTimer(quiz.questions[0]?.timeLimit || 15);  
  };  
  
  const handleNextQuestion = async (answerIndex) => {  
  const handleNextQuestion = async (answerIndex) => {  
    setSelectedAnswerIndex(answerIndex);  
    setSelectedAnswerIndex(answerIndex);  
    const currentQ = activeQuiz.questions[currentQuestionIndex];  
      
    let addedScore = 0;  
    let addedScore = 0;  
    if (answerIndex === currentQ.correctIndex) {  
    if (answerIndex === currentQ.correctIndex) {  
      addedScore = 100 + Math.round(quizTimer * 5);  
      addedScore = 100 + Math.round(quizTimer * 5);  
    }  
  
    setQuizScore(prev => prev + addedScore);  
  
    setTimeout(async () => {  
    setTimeout(async () => {  
      if (currentQuestionIndex + 1 < activeQuiz.questions.length) {  
      if (currentQuestionIndex + 1 < activeQuiz.questions.length) {  
        setCurrentQuestionIndex(prev => prev + 1);  
        setCurrentQuestionIndex(prev => prev + 1);  
        setSelectedAnswerIndex(null);  
        setSelectedAnswerIndex(null);  
        setQuizTimer(activeQuiz.questions[currentQuestionIndex + 1]?.timeLimit || 15);  
        setQuizTimer(activeQuiz.questions[currentQuestionIndex + 1]?.timeLimit || 15);  
      } else {  
        setQuizFinished(true);  
        try {  
          const scoreCollection = collection(db, 'artifacts', appId, 'public', 'data', 'quiz_scores');  
          await addDoc(scoreCollection, {  
            player: user.memberId,  
            quizTitle: activeQuiz.title,  
            score: quizScore + addedScore,  
            score: quizScore + addedScore,  
            timestamp: Date.now()  
            timestamp: Date.now()  
          });  
        } catch (err) {  
          console.error(err);  
          console.error(err);  
        }  
      }  
      }  
    }, 1200);  
  };  
  
  const handleDeleteMessage = async (msgId) => {  
  const handleDeleteMessage = async (msgId) => {  
    if (user.memberId !== SECRET_ADMIN) return;  
    if (user.memberId !== SECRET_ADMIN) return;  
    try {  
      const msgDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'chats', msgId);  
      const msgDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'chats', msgId);  
      await deleteDoc(msgDocRef);  
      await deleteDoc(msgDocRef);  
    } catch (err) {  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
  };  
  
  const handleDeletePost = async (postId) => {  
  const handleDeletePost = async (postId) => {  
    if (user.memberId !== SECRET_ADMIN && posts.find(p => p.id === postId)?.author !== user.memberId) return;  
    if (user.memberId !== SECRET_ADMIN && posts.find(p => p.id === postId)?.author !== user.memberId) return;  
    try {  
    try {  
      const postDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'posts', postId);  
      await deleteDoc(postDocRef);  
    } catch (err) {  
      console.error(err);  
    }  
    }  
  };  
  };  
  
  const toggleBanUser = async (memberId) => {  
    if (user.memberId !== SECRET_ADMIN) return;  
    const currentBan = memberStatus[memberId]?.isBanned || false;  
    const currentBan = memberStatus[memberId]?.isBanned || false;  
    try {  
    try {  
      const statusDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'status', memberId);  
      await setDoc(statusDocRef, { isBanned: !currentBan }, { merge: true });  
    } catch (err) {  
    } catch (err) {  
      console.error(err);  
      console.error(err);  
    }  
  };  
  };  
  
  const getActiveRoomDetails = () => {  
  const getActiveRoomDetails = () => {  
    if (activeRoomId === "global") {  
      return { name: "大廳公共廣播 📢", participants: CLASS_MEMBERS };  
      return { name: "大廳公共廣播 📢", participants: CLASS_MEMBERS };  
    }  
    if (activeRoomId.startsWith("private-omp-")) {  
      return { name: "Open Mind Pro 🤖", participants: [user.memberId, "Open Mind Pro"] };  
    }  
    }  
    const r = rooms.find(room => room.id === activeRoomId);  
    return r || { name: "班級討論室", participants: [] };  
  };  
  };  
  
  const activeRoom = getActiveRoomDetails();  
  const activeRoom = getActiveRoomDetails();  
  const myRooms = rooms.filter(r => r.participants?.includes(user.memberId));  
  const myRooms = rooms.filter(r => r.participants?.includes(user.memberId));  
  const filteredMessages = messages.filter(m => m.roomId === activeRoomId);  
  const filteredMessages = messages.filter(m => m.roomId === activeRoomId);  
  
  if (!isJoined || !user) {  
    return (  
    return (  
      <div className="min-h-screen bg-slate-950 flex flex-col justify-center items-center px-4 relative overflow-hidden">  
      <div className="min-h-screen bg-slate-950 flex flex-col justify-center items-center px-4 relative overflow-hidden">  
        <div className="absolute top-1/4 left-1/4 w-72 h-72 bg-emerald-500/10 rounded-full blur-3xl pointer-events-none"></div>  
        <div className="absolute bottom-1/4 right-1/4 w-72 h-72 bg-pink-500/10 rounded-full blur-3xl pointer-events-none"></div>  
        <div className="absolute bottom-1/4 right-1/4 w-72 h-72 bg-pink-500/10 rounded-full blur-3xl pointer-events-none"></div>  
  
        <div className="bg-slate-900 border border-slate-800 text-white rounded-3xl p-8 shadow-2xl max-w-md w-full relative z-10">  
          <div className="flex flex-col items-center mb-8">  
          <div className="flex flex-col items-center mb-8">  
            <div className="w-20 h-20 bg-gradient-to-tr from-emerald-500 to-teal-400 rounded-2xl flex items-center justify-center text-4xl mb-4 shadow-lg shadow-emerald-500/20">  
            <div className="w-20 h-20 bg-gradient-to-tr from-emerald-500 to-teal-400 rounded-2xl flex items-center justify-center text-4xl mb-4 shadow-lg shadow-emerald-500/20">  
              🎓  
              🎓  
            </div>  
            </div>  
            <h1 className="text-2xl font-black tracking-wider text-emerald-400 bg-gradient-to-r from-emerald-400 to-teal-300 bg-clip-text text-transparent">  
              6A 班級全能互聯網  
              6A 班級全能互聯網  
            </h1>  
            </h1>  
            <p className="text-slate-400 text-xs mt-2 font-medium tracking-wide">  
            <p className="text-slate-400 text-xs mt-2 font-medium tracking-wide">  
              免密碼快速通報入口 • 支持私人密聊及 Open Mind Pro AI  
              免密碼快速通報入口 • 支持私人密聊及 Open Mind Pro AI  
            </p>  
            </p>  
          </div>  
          </div>  
  
          <form onSubmit={handleLogin} className="space-y-6">  
            <div>  
            <div>  
              <label className="block text-xs font-bold uppercase tracking-wider mb-2 text-slate-300">  
              <label className="block text-xs font-bold uppercase tracking-wider mb-2 text-slate-300">  
                選擇您的身份 / 學號  
              </label>  
              <select  
                className="w-full bg-slate-800 border border-slate-700 rounded-xl py-3.5 px-4 text-white focus:outline-none focus:ring-2 focus:ring-emerald-500 transition-all text-sm font-semibold cursor-pointer"  
                value={selectedMember}  
                value={selectedMember}  
                onChange={(e) => setSelectedMember(e.target.value)}  
                required  
              >  
              >  
                <option value="">-- 請點選成員列表 --</option>  
                <option value="">-- 請點選成員列表 --</option>  
                <optgroup label="全班同學">  
                  {CLASS_MEMBERS.map(m => (  
                  {CLASS_MEMBERS.map(m => (  
                    <option key={m} value={m}>{m} 同學</option>  
                  ))}  
                  ))}  
                </optgroup>  
                <optgroup label="學校教職員">  
                <optgroup label="學校教職員">  
                  {TEACHERS.map(t => (  
                    <option key={t.id} value={t.name}>{t.name} ({t.role})</option>  
                    <option key={t.id} value={t.name}>{t.name} ({t.role})</option>  
                  ))}  
                  ))}  
                </optgroup>  
              </select>  
            </div>  
  
            <button  
              type="submit"  
              disabled={loading}  
              disabled={loading}  
              className="w-full bg-gradient-to-r from-emerald-500 to-teal-500 hover:from-emerald-400 hover:to-teal-400 active:scale-95 text-white font-bold py-4 px-4 rounded-xl shadow-lg shadow-emerald-500/20 transition-all duration-150 text-sm"  
              className="w-full bg-gradient-to-r from-emerald-500 to-teal-500 hover:from-emerald-400 hover:to-teal-400 active:scale-95 text-white font-bold py-4 px-4 rounded-xl shadow-lg shadow-emerald-500/20 transition-all duration-150 text-sm"  
            >  
              一鍵登入大廳  
              一鍵登入大廳  
            </button>  
          </form>  
          </form>  
  
          <div className="mt-8 pt-4 border-t border-slate-800 text-center text-[10px] text-slate-500 font-bold leading-relaxed">  
          <div className="mt-8 pt-4 border-t border-slate-800 text-center text-[10px] text-slate-500 font-bold leading-relaxed">  
            自動屏蔽粗言穢語機制已啟用 • 維護所有學生隱私保護  
          </div>  
          </div>  
        </div>  
        </div>  
      </div>  
      </div>  
    );  
    );  
  }  
  }  
  
  return (  
  return (  
    <div className="min-h-screen bg-slate-950 text-white flex flex-col font-sans select-none">  
        
      {/* 頂部主導航欄 */}  
      {/* 頂部主導航欄 */}  
      <header className="bg-slate-900/90 backdrop-blur-md border-b border-slate-800 sticky top-0 z-40 px-4 py-3 flex justify-between items-center shadow-xl">  
      <header className="bg-slate-900/90 backdrop-blur-md border-b border-slate-800 sticky top-0 z-40 px-4 py-3 flex justify-between items-center shadow-xl">  
        <div className="flex items-center space-x-3">  
        <div className="flex items-center space-x-3">  
          <div className="w-10 h-10 bg-gradient-to-tr from-emerald-500 to-indigo-500 rounded-xl flex items-center justify-center font-black text-lg shadow-inner">  
          <div className="w-10 h-10 bg-gradient-to-tr from-emerald-500 to-indigo-500 rounded-xl flex items-center justify-center font-black text-lg shadow-inner">  
            6A  
            6A  
          </div>  
          </div>  
          <div>  
          <div>  
            <h1 className="font-extrabold text-sm tracking-tight bg-gradient-to-r from-emerald-400 to-teal-300 bg-clip-text text-transparent">  
            <h1 className="font-extrabold text-sm tracking-tight bg-gradient-to-r from-emerald-400 to-teal-300 bg-clip-text text-transparent">  
              6A 班級聯絡與分享平台  
              6A 班級聯絡與分享平台  
            </h1>  
            <p className="text-xs text-slate-400">登入身份: <span className="text-emerald-400 font-bold">{user.memberId}</span></p>  
            <p className="text-xs text-slate-400">登入身份: <span className="text-emerald-400 font-bold">{user.memberId}</span></p>  
          </div>  
          </div>  
        </div>  
  
        {/* 桌面端分頁選單 */}  
        <div className="hidden lg:flex bg-slate-950 rounded-2xl p-1 border border-slate-800 text-xs gap-1">  
        <div className="hidden lg:flex bg-slate-950 rounded-2xl p-1 border border-slate-800 text-xs gap-1">  
          <button  
            onClick={() => setCurrentTab("chat")}  
            onClick={() => setCurrentTab("chat")}  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
              currentTab === "chat" ? "bg-emerald-500 text-white" : "text-slate-400 hover:text-white"  
            }`}  
          >  
            💬 密聊聊天室  
          </button>  
          <button  
            onClick={() => setCurrentTab("instagram")}  
            onClick={() => setCurrentTab("instagram")}  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
              currentTab === "instagram" ? "bg-pink-500 text-white" : "text-slate-400 hover:text-white"  
            }`}  
          >  
          >  
            📸 動態分享  
            📸 動態分享  
          </button>  
          </button>  
          <button  
            onClick={() => setCurrentTab("voting")}  
            onClick={() => setCurrentTab("voting")}  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
              currentTab === "voting" ? "bg-amber-500 text-slate-950" : "text-slate-400 hover:text-white"  
            }`}  
            }`}  
          >  
            🗳️ 功能投票  
          </button>  
          <button  
          <button  
            onClick={() => setCurrentTab("kahoot")}  
            onClick={() => setCurrentTab("kahoot")}  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
            className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
              currentTab === "kahoot" ? "bg-indigo-500 text-white" : "text-slate-400 hover:text-white"  
            }`}  
            }`}  
          >  
          >  
            🎮 Kahoot! 問答  
          </button>  
          </button>  
          {user.memberId === SECRET_ADMIN && (  
            <button  
              onClick={() => setCurrentTab("admin")}  
              onClick={() => setCurrentTab("admin")}  
              className={`px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${  
                currentTab === "admin" ? "bg-red-600 text-white" : "text-slate-400 hover:text-white"  
              }`}  
              }`}  
            >  
            >  
              🛠️ 班級維護  
            </button>  
            </button>  
          )}  
        </div>  
        </div>  
  
        <div className="flex items-center space-x-2">  
          <button  
          <button  
            onClick={handleLogout}  
            onClick={handleLogout}  
            className="hidden sm:block text-slate-400 hover:text-rose-400 hover:border-rose-500/40 text-xs bg-slate-850 px-4 py-2 rounded-xl border border-slate-800 transition font-bold"  
          >  
          >  
            退出  
            退出  
          </button>  
          </button>  
          <button   
          <button   
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}  
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}  
            className="lg:hidden p-2 text-slate-400 hover:text-white hover:bg-slate-800 rounded-xl transition"  
            className="lg:hidden p-2 text-slate-400 hover:text-white hover:bg-slate-800 rounded-xl transition"  
          >  
          >  
            {mobileMenuOpen ? <X className="w-5 h-5" /> : <Menu className="w-5 h-5" />}  
          </button>  
          </button>  
        </div>  
      </header>  
  
      {/* 行動端下拉選單 */}  
      {/* 行動端下拉選單 */}  
      {mobileMenuOpen && (  
      {mobileMenuOpen && (  
        <div className="lg:hidden bg-slate-900 border-b border-slate-800 p-4 space-y-2 text-sm z-30">  
          <button  
            onClick={() => { setCurrentTab("chat"); setMobileMenuOpen(false); }}  
            onClick={() => { setCurrentTab("chat"); setMobileMenuOpen(false); }}  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
              currentTab === "chat" ? "bg-emerald-500/10 text-emerald-400" : "text-slate-300"  
            }`}  
          >  
          >  
            <span>💬</span> <span>密聊聊天室</span>  
            <span>💬</span> <span>密聊聊天室</span>  
          </button>  
          <button  
          <button  
            onClick={() => { setCurrentTab("instagram"); setMobileMenuOpen(false); }}  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
              currentTab === "instagram" ? "bg-pink-500/10 text-pink-400" : "text-slate-300"  
            }`}  
          >  
            <span>📸</span> <span>動態分享</span>  
          </button>  
          <button  
            onClick={() => { setCurrentTab("voting"); setMobileMenuOpen(false); }}  
            onClick={() => { setCurrentTab("voting"); setMobileMenuOpen(false); }}  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
              currentTab === "voting" ? "bg-amber-500/10 text-amber-400" : "text-slate-300"  
            }`}  
          >  
            <span>🗳️</span> <span>功能投票</span>  
          </button>  
          </button>  
          <button  
          <button  
            onClick={() => { setCurrentTab("kahoot"); setMobileMenuOpen(false); }}  
            onClick={() => { setCurrentTab("kahoot"); setMobileMenuOpen(false); }}  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
            className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
              currentTab === "kahoot" ? "bg-indigo-500/10 text-indigo-400" : "text-slate-300"  
            }`}  
            }`}  
          >  
            <span>🎮</span> <span>Kahoot! 問答</span>  
            <span>🎮</span> <span>Kahoot! 問答</span>  
          </button>  
          </button>  
          {user.memberId === SECRET_ADMIN && (  
          {user.memberId === SECRET_ADMIN && (  
            <button  
              onClick={() => { setCurrentTab("admin"); setMobileMenuOpen(false); }}  
              className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
              className={`w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 ${  
                currentTab === "admin" ? "bg-red-500/10 text-red-400" : "text-slate-300"  
                currentTab === "admin" ? "bg-red-500/10 text-red-400" : "text-slate-300"  
              }`}  
            >  
              <span>🛠️</span> <span>班級維護</span>  
              <span>🛠️</span> <span>班級維護</span>  
            </button>  
            </button>  
          )}  
          <button  
            onClick={handleLogout}  
            onClick={handleLogout}  
            className="w-full text-left py-2.5 px-4 rounded-xl font-bold text-rose-400 border border-rose-500/20 bg-rose-500/5 flex items-center space-x-2"  
          >  
            <span>🚪</span> <span>登出帳號</span>  
            <span>🚪</span> <span>登出帳號</span>  
          </button>  
          </button>  
        </div>  
      )}  
  
      {/* 主體內容 */}  
      {/* 主體內容 */}  
      <main className="flex-1 flex flex-col overflow-hidden max-w-5xl w-full mx-auto relative">  
          
          
        {/* TAB 1: WhatsApp 密聊聊天室 */}  
        {currentTab === "chat" && (  
          <div className="flex-1 flex flex-col md:flex-row bg-slate-950 overflow-hidden h-full">  
              
              
            {/* 左側對話室與師長列表 */}  
            {/* 左側對話室與師長列表 */}  
            <div className="w-full md:w-80 bg-slate-900 border-b md:border-b-0 md:border-r border-slate-800 flex flex-col h-1/3 md:h-full overflow-y-auto p-3 shrink-0">  
              <div className="flex justify-between items-center mb-4">  
              <div className="flex justify-between items-center mb-4">  
                <h3 className="font-extrabold text-sm text-slate-200">我的聊天頻道 ({myRooms.length + 2})</h3>  
                <h3 className="font-extrabold text-sm text-slate-200">我的聊天頻道 ({myRooms.length + 2})</h3>  
                <button  
                  onClick={() => setShowCreateRoomModal(true)}  
                  onClick={() => setShowCreateRoomModal(true)}  
                  className="bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black text-xs py-1.5 px-3 rounded-lg shadow transition active:scale-95"  
                  className="bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black text-xs py-1.5 px-3 rounded-lg shadow transition active:scale-95"  
                >  
                  ➕ 新對話  
                  ➕ 新對話  
                </button>  
                </button>  
              </div>  
  
              {/* 聊天室頻道 */}  
              <div className="space-y-1.5 flex-1">  
              <div className="space-y-1.5 flex-1">  
                {/* 預設大廳 */}  
                <button  
                <button  
                  onClick={() => setActiveRoomId("global")}  
                  className={`w-full text-left p-3 rounded-xl transition flex items-center justify-between ${  
                  className={`w-full text-left p-3 rounded-xl transition flex items-center justify-between ${  
                    activeRoomId === "global"   
                    activeRoomId === "global"   
                      ? "bg-emerald-500/20 border border-emerald-500/40 text-emerald-300"   
                      ? "bg-emerald-500/20 border border-emerald-500/40 text-emerald-300"   
                      : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                      : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                  }`}  
                  }`}  
                >  
                >  
                  <div className="truncate">  
                  <div className="truncate">  
                    <span className="font-black text-xs block">📢 大廳公共廣播</span>  
                    <span className="text-[10px] text-slate-400">全班廣播對話</span>  
                    <span className="text-[10px] text-slate-400">全班廣播對話</span>  
                  </div>  
                  </div>  
                  <span className="text-xs shrink-0">🌍</span>  
                </button>  
  
                {/* 與 AI 的專屬聊天房 */}  
                {/* 與 AI 的專屬聊天房 */}  
                <button  
                  onClick={() => setActiveRoomId(`private-omp-${user.uid}`)}  
                  className={`w-full text-left p-3 rounded-xl transition flex items-center justify-between ${  
                    activeRoomId === `private-omp-${user.uid}`  
                    activeRoomId === `private-omp-${user.uid}`  
                      ? "bg-purple-500/20 border border-purple-500/40 text-purple-300"  
                      : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                      : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                  }`}  
                >  
                >  
                  <div className="truncate">  
                    <span className="font-black text-xs block">🤖 Open Mind Pro</span>  
                    <span className="text-[10px] text-slate-400">與維基百科 AI 私聊</span>  
                    <span className="text-[10px] text-slate-400">與維基百科 AI 私聊</span>  
                  </div>  
                  </div>  
                  <span className="text-xs shrink-0">✨</span>  
                </button>  
  
                {/* 學生/老師建置聊天 */}  
                {/* 學生/老師建置聊天 */}  
                {myRooms.map((room) => {  
                {myRooms.map((room) => {  
                  const isActive = activeRoomId === room.id;  
                  const isActive = activeRoomId === room.id;  
                  return (  
                  return (  
                    <button  
                    <button  
                      key={room.id}  
                      key={room.id}  
                      onClick={() => setActiveRoomId(room.id)}  
                      onClick={() => setActiveRoomId(room.id)}  
                      className={`w-full text-left p-3 rounded-xl transition flex items-center justify-between ${  
                      className={`w-full text-left p-3 rounded-xl transition flex items-center justify-between ${  
                        isActive   
                        isActive   
                          ? "bg-emerald-500/20 border border-emerald-500/40 text-emerald-300"   
                          : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                          : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"  
                      }`}  
                    >  
                      <div className="truncate pr-2">  
                        <span className="font-black text-xs block truncate">  
                          {room.isGroup ? "👥" : "👤"} {room.name}  
                          {room.isGroup ? "👥" : "👤"} {room.name}  
                        </span>  
                        <span className="text-[10px] text-slate-400 block truncate">  
                          成員: {room.participants?.join(", ")}  
                          成員: {room.participants?.join(", ")}  
                        </span>  
                        </span>  
                      </div>  
                    </button>  
                  );  
                  );  
                })}  
                })}  
              </div>  
              </div>  
  
              {/* 快速老師私人對話發起 */}  
              {/* 快速老師私人對話發起 */}  
              <div className="pt-4 border-t border-slate-800/60 mt-3">  
                <span className="text-[10px] font-bold text-slate-500 uppercase tracking-wider block mb-2 px-1">  
                <span className="text-[10px] font-bold text-slate-500 uppercase tracking-wider block mb-2 px-1">  
                  🏫 聯絡任教老師  
                  🏫 聯絡任教老師  
                </span>  
                <div className="space-y-1.5">  
                <div className="space-y-1.5">  
                  {TEACHERS.map((t) => (  
                    <button  
                      key={t.id}  
                      key={t.id}  
                      onClick={() => handleStartTeacherChat(t)}  
                      className="w-full text-left p-2.5 bg-slate-950/60 hover:bg-slate-800/40 border border-slate-850 rounded-xl transition flex items-center justify-between text-xs"  
                    >  
                      <span className="font-semibold text-slate-200">{t.name} ({t.role})</span>  
                      <span className="font-semibold text-slate-200">{t.name} ({t.role})</span>  
                      <span className="text-[10px] text-emerald-400">發起對話 💬</span>  
                    </button>  
                  ))}  
                  ))}  
                </div>  
                </div>  
              </div>  
            </div>  
  
            {/* 右側密聊核心 */}  
            {/* 右側密聊核心 */}  
            <div className="flex-1 flex flex-col bg-slate-900/40 relative h-2/3 md:h-full overflow-hidden">  
              <div className="absolute inset-0 opacity-5 pointer-events-none bg-[radial-gradient(#10b981_1px,transparent_1px)] [background-size:16px_16px]"></div>  
              <div className="absolute inset-0 opacity-5 pointer-events-none bg-[radial-gradient(#10b981_1px,transparent_1px)] [background-size:16px_16px]"></div>  
  
              {/* 聊天室標頭 */}  
              <div className="bg-slate-900/90 backdrop-blur border-b border-slate-800 p-3.5 flex justify-between items-center relative z-20">  
                <div className="truncate">  
                <div className="truncate">  
                  <h4 className="font-extrabold text-sm text-slate-100 truncate">  
                  <h4 className="font-extrabold text-sm text-slate-100 truncate">  
                    {activeRoom.name}  
                    {activeRoom.name}  
                  </h4>  
                  </h4>  
                  <p className="text-[10px] text-slate-400 truncate mt-0.5">  
                    參與者: {activeRoom.participants?.join(", ")}  
                  </p>  
                </div>  
              </div>  
  
              {/* 訊息展示區 */}  
              {/* 訊息展示區 */}  
              <div className="flex-1 overflow-y-auto p-4 space-y-4 relative z-10">  
                {filteredMessages.map((msg) => {  
                {filteredMessages.map((msg) => {  
                  const isMe = msg.sender === user.memberId;  
                  const isOmp = msg.sender === "Open Mind Pro 🤖";  
                  const isOmp = msg.sender === "Open Mind Pro 🤖";  
                  return (  
                  return (  
                    <div key={msg.id} className={`flex flex-col ${isMe ? 'items-end' : 'items-start'}`}>  
                    <div key={msg.id} className={`flex flex-col ${isMe ? 'items-end' : 'items-start'}`}>  
                      <div className="max-w-[85%] md:max-w-[75%]">  
                        <div className="flex items-center space-x-2 px-1 mb-1 text-[10px] text-slate-400">  
                        <div className="flex items-center space-x-2 px-1 mb-1 text-[10px] text-slate-400">  
                          <span className={`font-black ${isOmp ? 'text-purple-400' : 'text-emerald-400'}`}>  
                            {msg.sender}  
                          </span>  
                          <span>•</span>  
                          <span>{new Date(msg.createdAt).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</span>  
                        </div>  
                        </div>  
  
                        {/* 訊息氣泡 */}  
                        <div className={`p-3.5 rounded-2xl shadow-lg border text-sm ${  
                        <div className={`p-3.5 rounded-2xl shadow-lg border text-sm ${  
                          isMe   
                            ? 'bg-emerald-600/90 border-emerald-500/80 text-white rounded-tr-none'   
                            ? 'bg-emerald-600/90 border-emerald-500/80 text-white rounded-tr-none'   
                            : 'bg-slate-800 border-slate-700/80 text-slate-100 rounded-tl-none'  
                            : 'bg-slate-800 border-slate-700/80 text-slate-100 rounded-tl-none'  
                        }`}>  
                        }`}>  
                          {msg.text && <p className="whitespace-pre-wrap break-all leading-relaxed font-medium">{msg.text}</p>}  
  
                          {/* 本機上傳圖片 */}  
                          {msg.image && (  
                            <div className="mt-2 rounded-xl overflow-hidden border border-black/30 bg-slate-950 max-w-xs">  
                            <div className="mt-2 rounded-xl overflow-hidden border border-black/30 bg-slate-950 max-w-xs">  
                              <img src={msg.image} alt="上傳圖片" className="max-h-60 object-cover" />  
                              <img src={msg.image} alt="上傳圖片" className="max-h-60 object-cover" />  
                            </div>  
                            </div>  
                          )}  
                          )}  
  
                          {/* 網頁連結 */}  
                          {/* 網頁連結 */}  
                          {msg.link && (  
                            <a   
                            <a   
                              href={msg.link.url.startsWith("http") ? msg.link.url : `https://${msg.link.url}`}   
                              href={msg.link.url.startsWith("http") ? msg.link.url : `https://${msg.link.url}`}   
                              target="_blank"   
                              target="_blank"   
                              rel="noopener noreferrer"  
                              rel="noopener noreferrer"  
                              className="mt-2.5 block p-3 bg-slate-950/80 hover:bg-slate-950 border border-slate-700/60 rounded-xl transition duration-150"  
                            >  
                            >  
                              <div className="flex items-center justify-between text-xs text-emerald-400 font-bold mb-1">  
                              <div className="flex items-center justify-between text-xs text-emerald-400 font-bold mb-1">  
                                <span>🔗 {msg.link.title}</span>  
                                <span>🔗 {msg.link.title}</span>  
                                <span className="text-[10px] text-slate-500">外部連結 ↗</span>  
                              </div>  
                              </div>  
                              <span className="text-[11px] text-slate-400 block truncate">{msg.link.url}</span>  
                              <span className="text-[11px] text-slate-400 block truncate">{msg.link.url}</span>  
                            </a>  
                            </a>  
                          )}  
                          )}  
  
                          {/* HTML 應用 */}  
                          {/* HTML 應用 */}  
                          {msg.html && (  
                          {msg.html && (  
                            <div className="mt-2.5 p-3 bg-slate-950 rounded-xl border border-slate-800 flex flex-col space-y-2">  
                            <div className="mt-2.5 p-3 bg-slate-950 rounded-xl border border-slate-800 flex flex-col space-y-2">  
                              <div className="flex items-center justify-between text-xs text-purple-400 pb-1.5 border-b border-slate-800/80 font-bold">  
                              <div className="flex items-center justify-between text-xs text-purple-400 pb-1.5 border-b border-slate-800/80 font-bold">  
                                <span className="flex items-center space-x-1">  
                                <span className="flex items-center space-x-1">  
                                  <span>🌐</span>  
                                  <span>🌐</span>  
                                  <span>HTML 應用程式</span>  
                                  <span>HTML 應用程式</span>  
                                </span>  
                                </span>  
                              </div>  
                              <button  
                                onClick={() => setActiveHtmlPayload({ sender: msg.sender, code: msg.html })}  
                                onClick={() => setActiveHtmlPayload({ sender: msg.sender, code: msg.html })}  
                                className="w-full py-2 bg-purple-600 hover:bg-purple-500 text-white text-xs font-bold rounded-lg shadow-md transition active:scale-95"  
                                className="w-full py-2 bg-purple-600 hover:bg-purple-500 text-white text-xs font-bold rounded-lg shadow-md transition active:scale-95"  
                              >  
                                ▶️ 執行此應用程式  
                                ▶️ 執行此應用程式  
                              </button>  
                              </button>  
                            </div>  
                          )}  
                          )}  
                        </div>  
                        </div>  
  
                        {/* 靜默一鍵清除（僅 6A32 擁有，其他用戶完全看不到此按鈕） */}  
                        {user.memberId === SECRET_ADMIN && (  
                        {user.memberId === SECRET_ADMIN && (  
                          <button  
                          <button  
                            onClick={() => handleDeleteMessage(msg.id)}  
                            onClick={() => handleDeleteMessage(msg.id)}  
                            className="text-red-400 hover:text-red-300 text-[10px] mt-1 underline block ml-auto mr-1 font-bold"  
                            className="text-red-400 hover:text-red-300 text-[10px] mt-1 underline block ml-auto mr-1 font-bold"  
                          >  
                          >  
                            剷除言論  
                            剷除言論  
                          </button>  
                        )}  
                        )}  
                      </div>  
                      </div>  
                    </div>  
                  );  
                })}  
                })}  
                <div ref={chatEndRef} />  
                <div ref={chatEndRef} />  
              </div>  
              </div>  
  
              {/* 輸入欄區域 */}  
              <div className="bg-slate-900 border-t border-slate-800 p-3 space-y-2 relative z-10">  
              <div className="bg-slate-900 border-t border-slate-800 p-3 space-y-2 relative z-10">  
                <form onSubmit={handleSendMessage} className="space-y-2">  
                <form onSubmit={handleSendMessage} className="space-y-2">  
                  <div className="flex flex-wrap items-center gap-2">  
                  <div className="flex flex-wrap items-center gap-2">  
                    <label className="flex items-center space-x-1 bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs font-bold py-2 px-3 rounded-xl border border-slate-700 cursor-pointer transition active:scale-95">  
                      <ImageIcon className="w-3.5 h-3.5" />  
                      <ImageIcon className="w-3.5 h-3.5" />  
                      <span>{chatImageBase64 ? "已選擇相片" : "加入相片"}</span>  
                      <span>{chatImageBase64 ? "已選擇相片" : "加入相片"}</span>  
                      <input   
                        type="file"   
                        type="file"   
                        accept="image/*"   
                        accept="image/*"   
                        className="hidden"   
                        className="hidden"   
                        onChange={(e) => handleLocalImageUpload(e, setChatImageBase64)}  
                        onChange={(e) => handleLocalImageUpload(e, setChatImageBase64)}  
                      />  
                      />  
                    </label>  
                    </label>  
  
                    <button  
                      type="button"  
                      onClick={() => setShowLinkModal(true)}  
                      onClick={() => setShowLinkModal(true)}  
                      className={`flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border transition active:scale-95 ${  
                      className={`flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border transition active:scale-95 ${  
                        chatLinkUrl ? "bg-teal-600 border-teal-500 text-white" : "bg-slate-800 hover:bg-slate-700 text-slate-300 border-slate-700"  
                      }`}  
                    >  
                    >  
                      <LinkIcon className="w-3.5 h-3.5" />  
                      <LinkIcon className="w-3.5 h-3.5" />  
                      <span>{chatLinkUrl ? "已載入連結" : "上載連結"}</span>  
                      <span>{chatLinkUrl ? "已載入連結" : "上載連結"}</span>  
                    </button>  
                    </button>  
  
                    <button  
                      type="button"  
                      onClick={() => setShowHtmlModal(true)}  
                      className={`flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border transition active:scale-95 ${  
                      className={`flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border transition active:scale-95 ${  
                        htmlCode ? "bg-purple-600 border-purple-500 text-white" : "bg-slate-800 hover:bg-slate-700 text-slate-300 border-slate-700"  
                        htmlCode ? "bg-purple-600 border-purple-500 text-white" : "bg-slate-800 hover:bg-slate-700 text-slate-300 border-slate-700"  
                      }`}  
                      }`}  
                    >  
                      <Code className="w-3.5 h-3.5" />  
                      <Code className="w-3.5 h-3.5" />  
                      <span>{htmlCode ? "已附加網頁程式" : "附加網頁程式"}</span>  
                      <span>{htmlCode ? "已附加網頁程式" : "附加網頁程式"}</span>  
                    </button>  
                    </button>  
  
                    {(chatImageBase64 || chatLinkUrl || htmlCode) && (  
                    {(chatImageBase64 || chatLinkUrl || htmlCode) && (  
                      <button  
                        type="button"  
                        onClick={() => {  
                        onClick={() => {  
                          setChatImageBase64("");  
                          setChatImageBase64("");  
                          setChatLinkUrl("");  
                          setChatLinkUrl("");  
                          setChatLinkTitle("");  
                          setHtmlCode("");  
                          setHtmlCode("");  
                        }}  
                        className="text-[10px] text-rose-400 hover:underline font-bold"  
                      >  
                        清空附加內容  
                      </button>  
                      </button>  
                    )}  
                    )}  
                  </div>  
                  </div>  
  
                  <div className="flex items-center space-x-2">  
                  <div className="flex items-center space-x-2">  
                    <input  
                      type="text"  
                      type="text"  
                      placeholder="說點甚麼嗎... (自動屏蔽粗口言論)"  
                      className="flex-1 bg-slate-800 border border-slate-700 rounded-full py-2.5 px-4 text-white text-sm placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-emerald-500"  
                      className="flex-1 bg-slate-800 border border-slate-700 rounded-full py-2.5 px-4 text-white text-sm placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-emerald-500"  
                      value={newMessage}  
                      value={newMessage}  
                      onChange={(e) => setNewMessage(e.target.value)}  
                    />  
                    <button  
                    <button  
                      type="submit"  
                      type="submit"  
                      className="bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white p-3 rounded-full shadow-lg transition duration-150 flex items-center justify-center shrink-0"  
                    >  
                      <Send className="w-4 h-4" />  
                      <Send className="w-4 h-4" />  
                    </button>  
                    </button>  
                  </div>  
                </form>  
                </form>  
              </div>  
              </div>  
  
            </div>  
            </div>  
          </div>  
          </div>  
        )}  
        )}  
  
        {/* TAB 2: Instagram 動態分享 */}  
        {/* TAB 2: Instagram 動態分享 */}  
        {currentTab === "instagram" && (  
        {currentTab === "instagram" && (  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
            <div className="bg-slate-900 border border-slate-800 rounded-2xl p-4 flex items-center justify-between shadow-lg">  
            <div className="bg-slate-900 border border-slate-800 rounded-2xl p-4 flex items-center justify-between shadow-lg">  
              <div className="flex items-center space-x-3">  
                <div className="w-12 h-12 bg-gradient-to-tr from-yellow-400 via-pink-500 to-purple-600 rounded-full flex items-center justify-center text-xl font-black">  
                  📸  
                  📸  
                </div>  
                </div>  
                <div>  
                <div>  
                  <h4 className="font-extrabold text-sm text-slate-100">發布班級 Instagram Post！</h4>  
                  <p className="text-xs text-slate-400">在這裡上載本機相片、撰寫心得，分享日常生活。</p>  
                  <p className="text-xs text-slate-400">在這裡上載本機相片、撰寫心得，分享日常生活。</p>  
                </div>  
                </div>  
              </div>  
              </div>  
              <button  
                onClick={() => setShowCreatePost(true)}  
                onClick={() => setShowCreatePost(true)}  
                className="bg-pink-600 hover:bg-pink-500 active:scale-95 text-white text-xs font-bold py-2.5 px-4 rounded-full shadow-md transition"  
              >  
              >  
                📸 建立新貼文  
                📸 建立新貼文  
              </button>  
            </div>  
            </div>  
  
            {/* 新貼文面板 */}  
            {showCreatePost && (  
            {showCreatePost && (  
              <div className="bg-slate-900 border border-slate-700 rounded-2xl p-5 shadow-2xl space-y-4">  
              <div className="bg-slate-900 border border-slate-700 rounded-2xl p-5 shadow-2xl space-y-4">  
                <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
                  <h3 className="font-extrabold text-base text-pink-400">建立新動態貼文</h3>  
                  <h3 className="font-extrabold text-base text-pink-400">建立新動態貼文</h3>  
                  <button onClick={() => setShowCreatePost(false)} className="text-slate-400 hover:text-white">✕</button>  
                </div>  
                <form onSubmit={handleCreatePost} className="space-y-4">  
                <form onSubmit={handleCreatePost} className="space-y-4">  
                  <div>  
                    <label className="block text-xs font-bold mb-2 text-slate-300">選擇本機相片</label>  
                    <label className="block text-xs font-bold mb-2 text-slate-300">選擇本機相片</label>  
                    <div className="flex items-center space-x-3">  
                    <div className="flex items-center space-x-3">  
                      <label className="bg-slate-800 hover:bg-slate-700 border border-slate-700 rounded-xl px-4 py-2.5 text-xs font-bold cursor-pointer text-slate-200 transition">  
                        選擇圖片檔案  
                        選擇圖片檔案  
                        <input   
                          type="file"   
                          accept="image/*"   
                          className="hidden"   
                          onChange={(e) => handleLocalImageUpload(e, setNewPostImageBase64)}  
                          required  
                          required  
                        />  
                        />  
                      </label>  
                      </label>  
                      {newPostImageBase64 && (  
                      {newPostImageBase64 && (  
                        <div className="text-[10px] text-emerald-400 font-bold">  
                          ✓ 相片已載入並自動壓縮  
                          ✓ 相片已載入並自動壓縮  
                        </div>  
                      )}  
                      )}  
                    </div>  
                    </div>  
                    {newPostImageBase64 && (  
                    {newPostImageBase64 && (  
                      <div className="mt-3 rounded-xl overflow-hidden max-w-xs border border-slate-800">  
                        <img src={newPostImageBase64} alt="預覽" className="max-h-40 object-cover" />  
                      </div>  
                    )}  
                  </div>  
                  </div>  
  
                  <div>  
                  <div>  
                    <label className="block text-xs font-bold mb-1 text-slate-300">寫點感想...</label>  
                    <label className="block text-xs font-bold mb-1 text-slate-300">寫點感想...</label>  
                    <textarea  
                    <textarea  
                      placeholder="這刻在想什麼？分享寫給 6A 的大家..."  
                      placeholder="這刻在想什麼？分享寫給 6A 的大家..."  
                      rows="3"  
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm focus:outline-none focus:border-pink-500 resize-none"  
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm focus:outline-none focus:border-pink-500 resize-none"  
                      value={newPostText}  
                      value={newPostText}  
                      onChange={(e) => setNewPostText(e.target.value)}  
                      onChange={(e) => setNewPostText(e.target.value)}  
                      required  
                      required  
                    />  
                    />  
                  </div>  
                  </div>  
  
                  <div className="flex justify-end space-x-2 pt-2">  
                    <button  
                    <button  
                      type="button"  
                      onClick={() => setShowCreatePost(false)}  
                      onClick={() => setShowCreatePost(false)}  
                      className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                      className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                    >  
                    >  
                      取消  
                      取消  
                    </button>  
                    </button>  
                    <button  
                    <button  
                      type="submit"  
                      type="submit"  
                      className="px-5 py-2 bg-pink-600 hover:bg-pink-500 rounded-xl text-xs font-bold"  
                    >  
                      發布貼文  
                    </button>  
                    </button>  
                  </div>  
                </form>  
              </div>  
              </div>  
            )}  
  
            {/* 貼文牆列表 */}  
            {/* 貼文牆列表 */}  
            <div className="max-w-md mx-auto space-y-6">  
              {posts.map((post) => {  
              {posts.map((post) => {  
                const isLikedByMe = post.likes?.includes(user.memberId);  
                const isLikedByMe = post.likes?.includes(user.memberId);  
                return (  
                  <article key={post.id} className="bg-slate-900 border border-slate-800 rounded-2xl overflow-hidden shadow-xl">  
                    <div className="p-4 flex items-center justify-between">  
                    <div className="p-4 flex items-center justify-between">  
                      <div className="flex items-center space-x-3">  
                        <div className="w-10 h-10 rounded-full bg-gradient-to-tr from-yellow-400 via-pink-500 to-purple-600 p-0.5">  
                        <div className="w-10 h-10 rounded-full bg-gradient-to-tr from-yellow-400 via-pink-500 to-purple-600 p-0.5">  
                          <div className="w-full h-full rounded-full bg-slate-900 flex items-center justify-center text-xs font-black">  
                          <div className="w-full h-full rounded-full bg-slate-900 flex items-center justify-center text-xs font-black">  
                            {post.author}  
                          </div>  
                        </div>  
                        </div>  
                        <div>  
                          <span className="font-extrabold text-sm block text-slate-100">  
                          <span className="font-extrabold text-sm block text-slate-100">  
                            {post.author}  
                            {post.author}  
                          </span>  
                          <span className="text-[10px] text-slate-500 font-medium">  
                          <span className="text-[10px] text-slate-500 font-medium">  
                            {new Date(post.createdAt).toLocaleDateString()}  
                            {new Date(post.createdAt).toLocaleDateString()}  
                          </span>  
                          </span>  
                        </div>  
                      </div>  
  
                      {/* 靜默剷除（6A32 特權，但按鈕外觀與普通個人刪除一模一樣，隱蔽真實管理權） */}  
                      {/* 靜默剷除（6A32 特權，但按鈕外觀與普通個人刪除一模一樣，隱蔽真實管理權） */}  
                      {(user.memberId === SECRET_ADMIN || post.author === user.memberId) && (  
                        <button  
                        <button  
                          onClick={() => handleDeletePost(post.id)}  
                          className="text-slate-400 hover:text-red-400 text-xs bg-slate-800/80 p-2 rounded-xl transition"  
                        >  
                          <Trash className="w-4 h-4" />  
                          <Trash className="w-4 h-4" />  
                        </button>  
                        </button>  
                      )}  
                      )}  
                    </div>  
                    </div>  
  
                    <div className="aspect-square bg-slate-950 flex items-center justify-center overflow-hidden border-y border-slate-800/60">  
                    <div className="aspect-square bg-slate-950 flex items-center justify-center overflow-hidden border-y border-slate-800/60">  
                      <img  
                      <img  
                        src={post.image}  
                        src={post.image}  
                        alt="IG貼文"  
                        alt="IG貼文"  
                        className="w-full h-full object-cover"  
                        className="w-full h-full object-cover"  
                      />  
                    </div>  
  
                    <div className="p-4 space-y-2">  
                      <button  
                      <button  
                        onClick={() => handleLikePost(post.id, post.likes)}  
                        onClick={() => handleLikePost(post.id, post.likes)}  
                        className="flex items-center space-x-2 text-slate-300 hover:text-pink-500 transition group animate-bounce-short"  
                      >  
                        <Heart className={`w-5 h-5 ${isLikedByMe ? 'text-pink-500 fill-pink-500' : 'text-slate-400'}`} />  
                        <Heart className={`w-5 h-5 ${isLikedByMe ? 'text-pink-500 fill-pink-500' : 'text-slate-400'}`} />  
                        <span className="text-xs font-black">{post.likes?.length || 0} 個人覺得讚</span>  
                        <span className="text-xs font-black">{post.likes?.length || 0} 個人覺得讚</span>  
                      </button>  
  
                      <div className="text-sm leading-relaxed">  
                        <span className="font-black mr-2 text-pink-400">{post.author}</span>  
                        <span className="text-slate-200 whitespace-pre-wrap break-all font-medium">{post.text}</span>  
                        <span className="text-slate-200 whitespace-pre-wrap break-all font-medium">{post.text}</span>  
                      </div>  
                    </div>  
                  </article>  
                );  
                );  
              })}  
            </div>  
          </div>  
          </div>  
        )}  
        )}  
  
        {/* TAB 3: 功能建議投票 */}  
        {currentTab === "voting" && (  
        {currentTab === "voting" && (  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
            <div className="bg-gradient-to-r from-amber-500/20 to-orange-500/10 border border-amber-500/30 rounded-2xl p-5">  
            <div className="bg-gradient-to-r from-amber-500/20 to-orange-500/10 border border-amber-500/30 rounded-2xl p-5">  
              <h2 className="text-lg font-black text-amber-400 flex items-center space-x-2">  
              <h2 className="text-lg font-black text-amber-400 flex items-center space-x-2">  
                <span>🗳️</span>  
                <span>🗳️</span>  
                <span>6A 專屬班級民主功能提案區</span>  
                <span>6A 專屬班級民主功能提案區</span>  
              </h2>  
              <p className="text-xs text-slate-400 mt-1">  
                提出您的功能構思，讓全班同學一起投票。高票者將被採納！  
                提出您的功能構思，讓全班同學一起投票。高票者將被採納！  
              </p>  
              </p>  
            </div>  
            </div>  
  
            <div className="flex justify-between items-center">  
            <div className="flex justify-between items-center">  
              <h3 className="font-black text-sm text-slate-200">當前提案列表 ({polls.length})</h3>  
              <button  
              <button  
                onClick={() => setShowCreatePoll(true)}  
                onClick={() => setShowCreatePoll(true)}  
                className="bg-amber-500 hover:bg-amber-400 text-slate-950 font-extrabold text-xs py-2 px-4 rounded-xl shadow-lg transition active:scale-95"  
                className="bg-amber-500 hover:bg-amber-400 text-slate-950 font-extrabold text-xs py-2 px-4 rounded-xl shadow-lg transition active:scale-95"  
              >  
              >  
                💡 發起新功能提案  
                💡 發起新功能提案  
              </button>  
            </div>  
  
            {showCreatePoll && (  
              <div className="bg-slate-900 border border-slate-700 rounded-2xl p-5 shadow-2xl space-y-4">  
                <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
                <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
                  <h3 className="font-extrabold text-sm text-amber-400">發起一個功能建議</h3>  
                  <button onClick={() => setShowCreatePoll(false)} className="text-slate-400 hover:text-white">✕</button>  
                  <button onClick={() => setShowCreatePoll(false)} className="text-slate-400 hover:text-white">✕</button>  
                </div>  
                </div>  
                <form onSubmit={handleCreateProposal} className="space-y-3">  
                <form onSubmit={handleCreateProposal} className="space-y-3">  
                  <div>  
                  <div>  
                    <label className="block text-xs font-bold mb-1 text-slate-300">功能名稱</label>  
                    <label className="block text-xs font-bold mb-1 text-slate-300">功能名稱</label>  
                    <input  
                    <input  
                      type="text"  
                      type="text"  
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-sm focus:outline-none focus:border-amber-500"  
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-sm focus:outline-none focus:border-amber-500"  
                      value={newProposalTitle}  
                      value={newProposalTitle}  
                      onChange={(e) => setNewProposalTitle(e.target.value)}  
                      onChange={(e) => setNewProposalTitle(e.target.value)}  
                      required  
                    />  
                    />  
                  </div>  
                  </div>  
                  <div>  
                  <div>  
                    <label className="block text-xs font-bold mb-1 text-slate-300">功能詳細介紹</label>  
                    <textarea  
                    <textarea  
                      placeholder="簡述一下這個功能可以幫同學做甚麼..."  
                      rows="3"  
                      rows="3"  
                      className="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-xs focus:outline-none focus:border-amber-500 resize-none"  
                      value={newProposalDesc}  
                      onChange={(e) => setNewProposalDesc(e.target.value)}  
                      onChange={(e) => setNewProposalDesc(e.target.value)}  
                    />  
                    />  
                  </div>  
                  </div>  
                  <div className="flex justify-end space-x-2 pt-1">  
                  <div className="flex justify-end space-x-2 pt-1">  
                    <button  
                    <button  
                      type="button"  
                      type="button"  
                      onClick={() => setShowCreatePoll(false)}  
                      className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                      className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                    >  
                      取消  
                      取消  
                    </button>  
                    </button>  
                    <button  
                    <button  
                      type="submit"  
                      type="submit"  
                      className="px-5 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 rounded-xl text-xs font-extrabold"  
                      className="px-5 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 rounded-xl text-xs font-extrabold"  
                    >  
                      發起公投  
                      發起公投  
                    </button>  
                  </div>  
                </form>  
              </div>  
            )}  
  
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">  
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">  
              {polls.map((poll) => {  
              {polls.map((poll) => {  
                const totalVotes = poll.votes?.length || 0;  
                const totalVotes = poll.votes?.length || 0;  
                const hasVoted = poll.votes?.includes(user.memberId);  
                const percentage = Math.min(Math.round((totalVotes / 32) * 100), 100);  
                const percentage = Math.min(Math.round((totalVotes / 32) * 100), 100);  
  
                return (  
                return (  
                  <div key={poll.id} className="bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-lg">  
                  <div key={poll.id} className="bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-lg">  
                    <div className="flex justify-between items-start">  
                      <div>  
                        <span className="text-[10px] bg-slate-800 text-amber-400 px-2 py-0.5 rounded font-black">  
                          提案者: {poll.proposer}  
                          提案者: {poll.proposer}  
                        </span>  
                        <h4 className="font-extrabold text-base text-slate-100 mt-2">{poll.title}</h4>  
                        <h4 className="font-extrabold text-base text-slate-100 mt-2">{poll.title}</h4>  
                      </div>  
                      </div>  
                      <button  
                        onClick={() => handleVotePoll(poll.id, poll.votes)}  
                        className={`px-3 py-1.5 rounded-xl text-xs font-extrabold transition active:scale-95 flex items-center space-x-1 ${  
                          hasVoted   
                            ? "bg-amber-500 text-slate-950"   
                            : "bg-slate-800 hover:bg-slate-700 text-amber-400 border border-amber-500/30"  
                        }`}  
                      >  
                      >  
                        <ThumbsUp className="w-3.5 h-3.5" />  
                        <span>{hasVoted ? "已投票" : "投一票"}</span>  
                      </button>  
                      </button>  
                    </div>  
                    </div>  
  
                    <p className="text-xs text-slate-400 leading-relaxed font-medium">  
                    <p className="text-xs text-slate-400 leading-relaxed font-medium">  
                      {poll.description || "未提供詳細描述。"}  
                    </p>  
                    </p>  
  
                    <div className="space-y-1.5">  
                      <div className="flex justify-between text-[11px] font-bold text-slate-400">  
                      <div className="flex justify-between text-[11px] font-bold text-slate-400">  
                        <span>全班支持度 ({totalVotes}/32)</span>  
                        <span className="text-amber-400 font-extrabold">{percentage}%</span>  
                        <span className="text-amber-400 font-extrabold">{percentage}%</span>  
                      </div>  
                      </div>  
                      <div className="w-full bg-slate-950 h-2.5 rounded-full overflow-hidden border border-slate-800">  
                      <div className="w-full bg-slate-950 h-2.5 rounded-full overflow-hidden border border-slate-800">  
                        <div   
                        <div   
                          className="bg-gradient-to-r from-amber-500 to-orange-500 h-full rounded-full transition-all duration-500"   
                          className="bg-gradient-to-r from-amber-500 to-orange-500 h-full rounded-full transition-all duration-500"   
                          style={{ width: `${percentage}%` }}  
                          style={{ width: `${percentage}%` }}  
                        />  
                        />  
                      </div>  
                    </div>  
                    </div>  
                  </div>  
                );  
              })}  
              })}  
            </div>  
            </div>  
          </div>  
          </div>  
        )}  
        )}  
  
        {/* TAB 4: Kahoot! 問答 */}  
        {currentTab === "kahoot" && (  
        {currentTab === "kahoot" && (  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
            <div className="bg-gradient-to-r from-indigo-500/20 to-purple-500/10 border border-indigo-500/30 rounded-2xl p-5">  
              <h2 className="text-lg font-black text-indigo-400 flex items-center space-x-2">  
                <span>🎮</span>  
                <span>🎮</span>  
                <span>6A 班級問答王 (Kahoot! 版)</span>  
                <span>6A 班級問答王 (Kahoot! 版)</span>  
              </h2>  
              </h2>  
              <p className="text-xs text-slate-400 mt-1">  
              <p className="text-xs text-slate-400 mt-1">  
                答對題目、且速度越快得分越高！  
                答對題目、且速度越快得分越高！  
              </p>  
            </div>  
            </div>  
  
            {activeQuiz ? (  
              <div className="bg-slate-900 border border-indigo-500/40 rounded-3xl p-6 space-y-6 shadow-2xl relative overflow-hidden">  
                <div className="absolute -top-10 -right-10 w-40 h-40 bg-indigo-500/10 rounded-full blur-2xl"></div>  
  
                {!quizFinished ? (  
                  <>  
                  <>  
                    <div className="flex justify-between items-center pb-3 border-b border-slate-800">  
                    <div className="flex justify-between items-center pb-3 border-b border-slate-800">  
                      <div>  
                      <div>  
                        <span className="text-xs text-indigo-400 font-black tracking-wider uppercase">  
                          主題: {activeQuiz.title}  
                          主題: {activeQuiz.title}  
                        </span>  
                        <h4 className="text-xs text-slate-400 mt-1 font-bold">  
                        <h4 className="text-xs text-slate-400 mt-1 font-bold">  
                          問題數: {currentQuestionIndex + 1} / {activeQuiz.questions.length}  
                          問題數: {currentQuestionIndex + 1} / {activeQuiz.questions.length}  
                        </h4>  
                        </h4>  
                      </div>  
                        
                        
                      <div className="flex items-center space-x-2">  
                      <div className="flex items-center space-x-2">  
                        <span className="text-xs text-slate-400 font-bold">限時剩餘:</span>  
                        <span className={`text-xl font-black px-3 py-1 rounded-xl ${  
                          quizTimer <= 5 ? "bg-rose-500/20 text-rose-400 animate-bounce" : "bg-indigo-500/20 text-indigo-400"  
                        }`}>  
                        }`}>  
                          ⏱️ {quizTimer}s  
                          ⏱️ {quizTimer}s  
                        </span>  
                        </span>  
                      </div>  
                      </div>  
                    </div>  
                    </div>  
  
                    <div className="text-center py-6">  
                      <h3 className="text-xl font-extrabold text-slate-100 leading-relaxed">  
                        {activeQuiz.questions[currentQuestionIndex].question}  
                        {activeQuiz.questions[currentQuestionIndex].question}  
                      </h3>  
                    </div>  
                    </div>  
  
                    <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">  
                      {activeQuiz.questions[currentQuestionIndex].options.map((option, idx) => {  
                      {activeQuiz.questions[currentQuestionIndex].options.map((option, idx) => {  
                        const colors = [  
                        const colors = [  
                          "bg-rose-600 hover:bg-rose-500 text-white",  
                          "bg-rose-600 hover:bg-rose-500 text-white",  
                          "bg-blue-600 hover:bg-blue-500 text-white",  
                          "bg-blue-600 hover:bg-blue-500 text-white",  
                          "bg-yellow-500 hover:bg-yellow-400 text-slate-950",  
                          "bg-yellow-500 hover:bg-yellow-400 text-slate-950",  
                          "bg-emerald-600 hover:bg-emerald-500 text-white"  
                          "bg-emerald-600 hover:bg-emerald-500 text-white"  
                        ];  
                        ];  
                        const symbols = ["🔺", "🔷", "🟡", "🟩"];  
                        const symbols = ["🔺", "🔷", "🟡", "🟩"];  
  
                        const isAnswered = selectedAnswerIndex !== null;  
                        const isAnswered = selectedAnswerIndex !== null;  
                        const isCorrect = idx === activeQuiz.questions[currentQuestionIndex].correctIndex;  
                        const isMySelection = idx === selectedAnswerIndex;  
                        const isMySelection = idx === selectedAnswerIndex;  
  
                        let statusStyle = colors[idx];  
                        let statusStyle = colors[idx];  
                        if (isAnswered) {  
                          if (isCorrect) {  
                          if (isCorrect) {  
                            statusStyle = "bg-emerald-500 text-white ring-4 ring-emerald-300 animate-pulse";  
                          } else if (isMySelection) {  
                          } else if (isMySelection) {  
                            statusStyle = "bg-rose-500 text-white opacity-90 ring-4 ring-rose-300";  
                            statusStyle = "bg-rose-500 text-white opacity-90 ring-4 ring-rose-300";  
                          } else {  
                          } else {  
                            statusStyle = "bg-slate-800 text-slate-500 opacity-30 cursor-not-allowed";  
                            statusStyle = "bg-slate-800 text-slate-500 opacity-30 cursor-not-allowed";  
                          }  
                        }  
  
                        return (  
                        return (  
                          <button  
                            key={idx}  
                            key={idx}  
                            disabled={isAnswered}  
                            disabled={isAnswered}  
                            onClick={() => handleNextQuestion(idx)}  
                            className={`p-4 rounded-2xl font-black text-sm flex items-center space-x-3 transition duration-150 transform active:scale-95 text-left ${statusStyle}`}  
                            className={`p-4 rounded-2xl font-black text-sm flex items-center space-x-3 transition duration-150 transform active:scale-95 text-left ${statusStyle}`}  
                          >  
                            <span className="text-base">{symbols[idx]}</span>  
                            <span className="text-base">{symbols[idx]}</span>  
                            <span className="flex-1">{option}</span>  
                          </button>  
                          </button>  
                        );  
                      })}  
                    </div>  
  
                    <div className="text-right text-xs text-slate-400 font-bold">  
                    <div className="text-right text-xs text-slate-400 font-bold">  
                      累積得分: <span className="text-indigo-400 text-sm font-black">{quizScore} 分</span>  
                    </div>  
                    </div>  
                  </>  
                  </>  
                ) : (  
                ) : (  
                  <div className="text-center py-8 space-y-5">  
                    <span className="text-6xl block">🏆</span>  
                    <span className="text-6xl block">🏆</span>  
                    <h3 className="text-2xl font-black text-indigo-400">恭喜完成挑戰！</h3>  
                    <h3 className="text-2xl font-black text-indigo-400">恭喜完成挑戰！</h3>  
                    <div className="text-4xl font-black text-yellow-400 tracking-wider">  
                      {quizScore} 分  
                    </div>  
                    <div className="pt-4">  
                    <div className="pt-4">  
                      <button  
                        onClick={() => setActiveQuiz(null)}  
                        className="bg-indigo-600 hover:bg-indigo-500 text-white font-extrabold text-xs py-3 px-6 rounded-xl shadow-lg transition active:scale-95"  
                        className="bg-indigo-600 hover:bg-indigo-500 text-white font-extrabold text-xs py-3 px-6 rounded-xl shadow-lg transition active:scale-95"  
                      >  
                      >  
                        返回問答大廳  
                        返回問答大廳  
                      </button>  
                      </button>  
                    </div>  
                    </div>  
                  </div>  
                  </div>  
                )}  
              </div>  
              </div>  
            ) : (  
              <div className="grid grid-cols-1 md:grid-cols-3 gap-6">  
                <div className="md:col-span-2 space-y-4">  
                <div className="md:col-span-2 space-y-4">  
                  <div className="flex justify-between items-center">  
                    <h3 className="font-extrabold text-sm text-slate-200">可供挑戰的問答題目 ({quizzes.length})</h3>  
                    <h3 className="font-extrabold text-sm text-slate-200">可供挑戰的問答題目 ({quizzes.length})</h3>  
                    <button  
                      onClick={() => setShowCreateQuiz(true)}  
                      onClick={() => setShowCreateQuiz(true)}  
                      className="bg-indigo-500 hover:bg-indigo-400 text-white font-extrabold text-xs py-2 px-3.5 rounded-xl transition active:scale-95"  
                    >  
                    >  
                      ➕ 新增自訂問答  
                    </button>  
                    </button>  
                  </div>  
                  </div>  
  
                  {showCreateQuiz && (  
                    <div className="bg-slate-900 border border-indigo-500/30 rounded-2xl p-5 space-y-4">  
                      <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
                        <h4 className="font-extrabold text-sm text-indigo-400">設計新 Kahoot! 問答</h4>  
                        <h4 className="font-extrabold text-sm text-indigo-400">設計新 Kahoot! 問答</h4>  
                        <button onClick={() => setShowCreateQuiz(false)} className="text-slate-400 hover:text-white">✕</button>  
                      </div>  
                      </div>  
  
                      <div className="space-y-4">  
                      <div className="space-y-4">  
                        <div>  
                          <label className="block text-xs font-bold mb-1 text-slate-300">問答主題名稱</label>  
                          <label className="block text-xs font-bold mb-1 text-slate-300">問答主題名稱</label>  
                          <input  
                          <input  
                            type="text"  
                            className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white focus:outline-none focus:border-indigo-500"  
                            value={newQuizTitle}  
                            value={newQuizTitle}  
                            onChange={(e) => setNewQuizTitle(e.target.value)}  
                            onChange={(e) => setNewQuizTitle(e.target.value)}  
                          />  
                          />  
                        </div>  
                        </div>  
  
                        {newQuizQuestions.map((q, qIdx) => (  
                        {newQuizQuestions.map((q, qIdx) => (  
                          <div key={qIdx} className="bg-slate-950 p-3.5 rounded-xl border border-slate-800 space-y-2.5">  
                          <div key={qIdx} className="bg-slate-950 p-3.5 rounded-xl border border-slate-800 space-y-2.5">  
                            <span className="text-[10px] bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded font-black">  
                              問題 {qIdx + 1}  
                            </span>  
                            <input  
                              type="text"  
                              type="text"  
                              placeholder="輸入問題本體"  
                              className="w-full bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs focus:outline-none text-white"  
                              className="w-full bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs focus:outline-none text-white"  
                              value={q.question}  
                              onChange={(e) => {  
                              onChange={(e) => {  
                                const newQs = [...newQuizQuestions];  
                                const newQs = [...newQuizQuestions];  
                                newQs[qIdx].question = e.target.value;  
                                newQs[qIdx].question = e.target.value;  
                                setNewQuizQuestions(newQs);  
                              }}  
                              }}  
                            />  
                              
                              
                            <div className="grid grid-cols-2 gap-2">  
                            <div className="grid grid-cols-2 gap-2">  
                              {q.options.map((opt, optIdx) => (  
                              {q.options.map((opt, optIdx) => (  
                                <input  
                                  key={optIdx}  
                                  type="text"  
                                  type="text"  
                                  placeholder={`選項 ${optIdx + 1}`}  
                                  placeholder={`選項 ${optIdx + 1}`}  
                                  className="bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs text-white focus:outline-none"  
                                  value={opt}  
                                  onChange={(e) => {  
                                    const newQs = [...newQuizQuestions];  
                                    newQs[qIdx].options[optIdx] = e.target.value;  
                                    newQs[qIdx].options[optIdx] = e.target.value;  
                                    setNewQuizQuestions(newQs);  
                                    setNewQuizQuestions(newQs);  
                                  }}  
                                  }}  
                                />  
                              ))}  
                            </div>  
                            </div>  
  
                            <div className="flex justify-between items-center pt-1.5">  
                            <div className="flex justify-between items-center pt-1.5">  
                              <div className="flex items-center space-x-2 text-xs">  
                                <span className="text-slate-400">正確答案選項:</span>  
                                <select  
                                  className="bg-slate-800 border border-slate-700 rounded p-1 text-xs"  
                                  value={q.correctIndex}  
                                  value={q.correctIndex}  
                                  onChange={(e) => {  
                                  onChange={(e) => {  
                                    const newQs = [...newQuizQuestions];  
                                    const newQs = [...newQuizQuestions];  
                                    newQs[qIdx].correctIndex = parseInt(e.target.value);  
                                    newQs[qIdx].correctIndex = parseInt(e.target.value);  
                                    setNewQuizQuestions(newQs);  
                                  }}  
                                  }}  
                                >  
                                  <option value={0}>選項 1</option>  
                                  <option value={1}>選項 2</option>  
                                  <option value={1}>選項 2</option>  
                                  <option value={2}>選項 3</option>  
                                  <option value={3}>選項 4</option>  
                                  <option value={3}>選項 4</option>  
                                </select>  
                              </div>  
                              </div>  
                            </div>  
                            </div>  
                          </div>  
                          </div>  
                        ))}  
  
                        <div className="flex justify-between pt-2">  
                        <div className="flex justify-between pt-2">  
                          <button  
                          <button  
                            type="button"  
                            onClick={() => setNewQuizQuestions([...newQuizQuestions, { question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }])}  
                            className="text-xs text-indigo-400 hover:underline font-bold"  
                            className="text-xs text-indigo-400 hover:underline font-bold"  
                          >  
                            ➕ 新增多一題問題  
                          </button>  
                            
                            
                          <div className="flex space-x-2">  
                            <button  
                            <button  
                              onClick={() => setShowCreateQuiz(false)}  
                              onClick={() => setShowCreateQuiz(false)}  
                              className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                              className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                            >  
                            >  
                              取消  
                              取消  
                            </button>  
                            </button>  
                            <button  
                            <button  
                              onClick={handleCreateQuiz}  
                              onClick={handleCreateQuiz}  
                              className="px-5 py-2 bg-indigo-600 hover:bg-indigo-500 rounded-xl text-xs font-black"  
                            >  
                            >  
                              儲存並發布  
                              儲存並發布  
                            </button>  
                            </button>  
                          </div>  
                          </div>  
                        </div>  
                      </div>  
                    </div>  
                  )}  
                  )}  
  
                  <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">  
                  <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">  
                    {quizzes.map((quiz) => (  
                    {quizzes.map((quiz) => (  
                      <div key={quiz.id} className="bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-md">  
                      <div key={quiz.id} className="bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-md">  
                        <div>  
                        <div>  
                          <span className="text-[10px] bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded font-black">  
                          <span className="text-[10px] bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded font-black">  
                            出題人: {quiz.creator}  
                            出題人: {quiz.creator}  
                          </span>  
                          <h4 className="font-extrabold text-base text-slate-200 mt-2">{quiz.title}</h4>  
                          <h4 className="font-extrabold text-base text-slate-200 mt-2">{quiz.title}</h4>  
                        </div>  
                        </div>  
                        <button  
                        <button  
                          onClick={() => handleStartQuiz(quiz)}  
                          onClick={() => handleStartQuiz(quiz)}  
                          className="w-full bg-indigo-600 hover:bg-indigo-500 active:scale-95 text-white font-extrabold text-xs py-2 px-4 rounded-xl shadow transition"  
                        >  
                          ⚡ 開始挑戰此問答  
                          ⚡ 開始挑戰此問答  
                        </button>  
                      </div>  
                      </div>  
                    ))}  
                    ))}  
                  </div>  
                  </div>  
                </div>  
                </div>  
  
                <div className="bg-slate-900 border border-slate-800 rounded-2xl p-4 h-fit">  
                  <h3 className="font-black text-sm text-yellow-400 flex items-center space-x-1.5 mb-3 pb-2 border-b border-slate-800">  
                  <h3 className="font-black text-sm text-yellow-400 flex items-center space-x-1.5 mb-3 pb-2 border-b border-slate-800">  
                    <span>👑</span>  
                    <span>班級問答英雄榜</span>  
                  </h3>  
                  </h3>  
                    
                    
                  <div className="space-y-2.5">  
                  <div className="space-y-2.5">  
                    {quizScores.slice(0, 10).map((score, index) => {  
                    {quizScores.slice(0, 10).map((score, index) => {  
                      const medal = index === 0 ? "🥇" : index === 1 ? "🥈" : index === 2 ? "🥉" : `${index + 1}.`;  
                      return (  
                      return (  
                        <div key={score.id} className="flex justify-between items-center bg-slate-950 p-2.5 rounded-xl border border-slate-800 text-xs">  
                        <div key={score.id} className="flex justify-between items-center bg-slate-950 p-2.5 rounded-xl border border-slate-800 text-xs">  
                          <div className="flex items-center space-x-2">  
                            <span className="font-black text-sm text-yellow-500">{medal}</span>  
                            <span className="font-black text-sm text-yellow-500">{medal}</span>  
                            <div>  
                              <span className="font-black text-slate-200 block">{score.player}</span>  
                              <span className="text-[9px] text-slate-500 block truncate max-w-[120px]">  
                              <span className="text-[9px] text-slate-500 block truncate max-w-[120px]">  
                                {score.quizTitle}  
                              </span>  
                            </div>  
                            </div>  
                          </div>  
                          </div>  
                          <span className="font-black text-indigo-400">{score.score} 分</span>  
                          <span className="font-black text-indigo-400">{score.score} 分</span>  
                        </div>  
                        </div>  
                      );  
                      );  
                    })}  
                    })}  
                  </div>  
                </div>  
              </div>  
            )}  
          </div>  
          </div>  
        )}  
  
        {/* TAB 5: 隱藏維護 (僅 6A32 登入時會出現在選單中，名稱為無害的 "班級統計維護") */}  
        {currentTab === "admin" && user.memberId === SECRET_ADMIN && (  
        {currentTab === "admin" && user.memberId === SECRET_ADMIN && (  
          <div className="flex-1 overflow-y-auto p-4 space-y-6">  
            <div className="bg-red-500/10 border border-red-500/30 rounded-2xl p-5 shadow-lg">  
            <div className="bg-red-500/10 border border-red-500/30 rounded-2xl p-5 shadow-lg">  
              <h2 className="text-lg font-bold text-red-400 flex items-center space-x-2">  
                <span>🛡️</span>  
                <span>班級維護管理通道 (隱形特權)</span>  
              </h2>  
              <p className="text-xs text-slate-400 mt-1">  
              <p className="text-xs text-slate-400 mt-1">  
                此處僅 6A32 帳號能夠瀏覽。您可以悄悄在此針對同學實施禁言。被禁言的同學將無法發言或發表貼文。  
              </p>  
              </p>  
            </div>  
            </div>  
  
            <div className="bg-slate-900 border border-slate-800 rounded-2xl p-4">  
              <h3 className="font-bold text-sm mb-4 pb-2 border-b border-slate-800">班級成員禁言維護列表</h3>  
              <h3 className="font-bold text-sm mb-4 pb-2 border-b border-slate-800">班級成員禁言維護列表</h3>  
                
              <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">  
                {CLASS_MEMBERS.filter(member => member !== SECRET_ADMIN).map((member) => {  
                {CLASS_MEMBERS.filter(member => member !== SECRET_ADMIN).map((member) => {  
                  const isUserBanned = memberStatus[member]?.isBanned || false;  
                  return (  
                    <div   
                      key={member}   
                      key={member}   
                      className={`p-3 rounded-xl border flex items-center justify-between transition ${  
                        isUserBanned ? 'bg-red-950/20 border-red-900/60' : 'bg-slate-800/50 border-slate-700/60'  
                      }`}  
                      }`}  
                    >  
                    >  
                      <div className="flex items-center space-x-2">  
                      <div className="flex items-center space-x-2">  
                        <div className={`w-8 h-8 rounded-full flex items-center justify-center text-xs font-bold ${  
                        <div className={`w-8 h-8 rounded-full flex items-center justify-center text-xs font-bold ${  
                          isUserBanned ? 'bg-red-600 text-white' : 'bg-emerald-600 text-white'  
                          isUserBanned ? 'bg-red-600 text-white' : 'bg-emerald-600 text-white'  
                        }`}>  
                          {member[2]+member[3]}  
                          {member[2]+member[3]}  
                        </div>  
                        <div>  
                        <div>  
                          <span className="font-bold text-sm text-slate-100">{member}</span>  
                        </div>  
                      </div>  
                      </div>  
  
                      <button  
                      <button  
                        onClick={() => toggleBanUser(member)}  
                        className={`text-xs px-2.5 py-1.5 rounded-lg font-bold transition ${  
                        className={`text-xs px-2.5 py-1.5 rounded-lg font-bold transition ${  
                          isUserBanned   
                            ? 'bg-emerald-500 hover:bg-emerald-600 text-slate-950 shadow-md'   
                            : 'bg-red-500/20 hover:bg-red-500/30 text-red-400 border border-red-500/30'  
                        }`}  
                        }`}  
                      >  
                      >  
                        {isUserBanned ? "解禁" : "禁言"}  
                        {isUserBanned ? "解禁" : "禁言"}  
                      </button>  
                      </button>  
                    </div>  
                    </div>  
                  );  
                })}  
                })}  
              </div>  
            </div>  
            </div>  
          </div>  
          </div>  
        )}  
  
      </main>  
  
      {/* MODALS */}  
      {/* MODALS */}  
      {/* 建立私人聊天對話 */}  
      {showCreateRoomModal && (  
      {showCreateRoomModal && (  
        <div className="fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center z-50 p-4">  
          <div className="bg-slate-900 border border-slate-700 rounded-3xl w-full max-w-md p-5 shadow-2xl space-y-4">  
          <div className="bg-slate-900 border border-slate-700 rounded-3xl w-full max-w-md p-5 shadow-2xl space-y-4">  
            <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
              <h3 className="font-extrabold text-base text-emerald-400 flex items-center space-x-2">  
              <h3 className="font-extrabold text-base text-emerald-400 flex items-center space-x-2">  
                <span>💬</span>  
                <span>💬</span>  
                <span>開啟新私人對話 / 小組群組</span>  
                <span>開啟新私人對話 / 小組群組</span>  
              </h3>  
              </h3>  
              <button onClick={() => setShowCreateRoomModal(false)} className="text-slate-400 hover:text-white">✕</button>  
            </div>  
            </div>  
  
            <form onSubmit={handleCreateRoom} className="space-y-4">  
              <div>  
              <div>  
                <label className="block text-xs font-bold mb-1 text-slate-300">小組群組名稱 (選填)</label>  
                <label className="block text-xs font-bold mb-1 text-slate-300">小組群組名稱 (選填)</label>  
                <input  
                  type="text"  
                  placeholder="例如：6A 專題報告小組"  
                  className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white focus:outline-none"  
                  className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white focus:outline-none"  
                  value={newGroupName}  
                  value={newGroupName}  
                  onChange={(e) => setNewGroupName(e.target.value)}  
                  onChange={(e) => setNewGroupName(e.target.value)}  
                />  
                />  
              </div>  
  
              <div>  
              <div>  
                <label className="block text-xs font-bold mb-2 text-slate-300">選擇參與的同學 (可多選群聊)</label>  
                <div className="grid grid-cols-3 gap-2 max-h-48 overflow-y-auto p-2 bg-slate-950 rounded-xl border border-slate-800">  
                <div className="grid grid-cols-3 gap-2 max-h-48 overflow-y-auto p-2 bg-slate-950 rounded-xl border border-slate-800">  
                  {CLASS_MEMBERS.filter(m => m !== user.memberId).map((member) => {  
                  {CLASS_MEMBERS.filter(m => m !== user.memberId).map((member) => {  
                    const isSelected = newRoomParticipants.includes(member);  
                    const isSelected = newRoomParticipants.includes(member);  
                    return (  
                    return (  
                      <button  
                      <button  
                        type="button"  
                        key={member}  
                        key={member}  
                        onClick={() => {  
                        onClick={() => {  
                          setNewRoomParticipants(prev =>   
                          setNewRoomParticipants(prev =>   
                            prev.includes(member) ? prev.filter(id => id !== member) : [...prev, member]  
                          );  
                          );  
                        }}  
                        }}  
                        className={`py-1.5 px-2 rounded-lg text-xs font-bold border transition ${  
                        className={`py-1.5 px-2 rounded-lg text-xs font-bold border transition ${  
                          isSelected   
                          isSelected   
                            ? "bg-emerald-500/20 border-emerald-500 text-emerald-400"   
                            : "bg-slate-800/40 border-slate-700/40 text-slate-400 hover:bg-slate-800"  
                            : "bg-slate-800/40 border-slate-700/40 text-slate-400 hover:bg-slate-800"  
                        }`}  
                        }`}  
                      >  
                      >  
                        {member}  
                        {member}  
                      </button>  
                      </button>  
                    );  
                    );  
                  })}  
                </div>  
              </div>  
  
              <div className="flex justify-end space-x-2 pt-2">  
                <button  
                  type="button"  
                  type="button"  
                  onClick={() => {  
                  onClick={() => {  
                    setShowCreateRoomModal(false);  
                    setNewRoomParticipants([]);  
                    setNewRoomParticipants([]);  
                    setNewGroupName("");  
                  }}  
                  }}  
                  className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                  className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
                >  
                >  
                  取消  
                </button>  
                </button>  
                <button  
                <button  
                  type="submit"  
                  className="px-5 py-2 bg-emerald-500 hover:bg-emerald-400 text-slate-950 rounded-xl text-xs font-black shadow"  
                  className="px-5 py-2 bg-emerald-500 hover:bg-emerald-400 text-slate-950 rounded-xl text-xs font-black shadow"  
                >  
                >  
                  立即建立  
                </button>  
              </div>  
              </div>  
            </form>  
            </form>  
          </div>  
          </div>  
        </div>  
        </div>  
      )}  
  
      {/* HTML 附加 Modal */}  
      {showHtmlModal && (  
        <div className="fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center z-50 p-4">  
        <div className="fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center z-50 p-4">  
          <div className="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-xl p-5 shadow-2xl space-y-4">  
            <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
            <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
              <h3 className="font-extrabold text-base text-purple-400 flex items-center space-x-2">  
              <h3 className="font-extrabold text-base text-purple-400 flex items-center space-x-2">  
                <span>💻</span>  
                <span>附加/上傳 HTML 互動程式</span>  
                <span>附加/上傳 HTML 互動程式</span>  
              </h3>  
              </h3>  
              <button onClick={() => setShowHtmlModal(false)} className="text-slate-400 hover:text-white">✕</button>  
            </div>  
            </div>  
  
            <div className="space-y-3">  
            <div className="space-y-3">  
              <textarea  
              <textarea  
                placeholder="<!DOCTYPE html><html><body><h3>應用程式範例</h3></body></html>"  
                placeholder="<!DOCTYPE html><html><body><h3>應用程式範例</h3></body></html>"  
                rows="9"  
                className="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-xs font-mono text-emerald-400 focus:outline-none focus:border-purple-500 resize-none"  
                value={htmlCode}  
                value={htmlCode}  
                onChange={(e) => setHtmlCode(e.target.value)}  
              />  
              />  
            </div>  
  
            <div className="flex justify-end space-x-2 pt-2">  
              <button  
              <button  
                type="button"  
                type="button"  
                onClick={() => setShowHtmlModal(false)}  
                onClick={() => setShowHtmlModal(false)}  
                className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
              >  
              >  
                取消  
              </button>  
              <button  
                onClick={() => {  
                onClick={() => {  
                  if (htmlCode.trim()) {  
                  if (htmlCode.trim()) {  
                    setShowHtmlModal(false);  
                    setShowHtmlModal(false);  
                  } else {  
                  } else {  
                    alert("請輸入內容！");  
                    alert("請輸入內容！");  
                  }  
                }}  
                }}  
                className="px-5 py-2 bg-purple-600 hover:bg-purple-500 rounded-xl text-xs font-bold text-white shadow"  
                className="px-5 py-2 bg-purple-600 hover:bg-purple-500 rounded-xl text-xs font-bold text-white shadow"  
              >  
              >  
                確認附加  
                確認附加  
              </button>  
              </button>  
            </div>  
            </div>  
          </div>  
        </div>  
        </div>  
      )}  
  
      {/* 上載分享連結 Modal */}  
      {showLinkModal && (  
      {showLinkModal && (  
        <div className="fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center z-50 p-4">  
        <div className="fixed inset-0 bg-black/85 backdrop-blur-sm flex items-center justify-center z-50 p-4">  
          <div className="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-md p-5 shadow-2xl space-y-4">  
          <div className="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-md p-5 shadow-2xl space-y-4">  
            <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
            <div className="flex justify-between items-center pb-2 border-b border-slate-800">  
              <h3 className="font-extrabold text-base text-teal-400 flex items-center space-x-2">  
                <span>🔗</span>  
                <span>上載網頁分享連結</span>  
                <span>上載網頁分享連結</span>  
              </h3>  
              </h3>  
              <button onClick={() => setShowLinkModal(false)} className="text-slate-400 hover:text-white">✕</button>  
              <button onClick={() => setShowLinkModal(false)} className="text-slate-400 hover:text-white">✕</button>  
            </div>  
            </div>  
  
            <div className="space-y-3">  
              <div>  
              <div>  
                <label className="block text-xs font-bold mb-1 text-slate-300">連結標題</label>  
                <input  
                <input  
                  type="text"  
                  className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs"  
                  className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs"  
                  value={chatLinkTitle}  
                  onChange={(e) => setChatLinkTitle(e.target.value)}  
                  onChange={(e) => setChatLinkTitle(e.target.value)}  
                  placeholder="輸入連結標題描述..."  
                />  
                />  
              </div>  
              </div>  
              <div>  
              <div>  
                <label className="block text-xs font-bold mb-1 text-slate-300">連結 URL</label>  
                <label className="block text-xs font-bold mb-1 text-slate-300">連結 URL</label>  
                <input  
                  type="text"  
                  className="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs"  
                  value={chatLinkUrl}  
                  value={chatLinkUrl}  
                  onChange={(e) => setChatLinkUrl(e.target.value)}  
                  placeholder="www.example.com"  
                  placeholder="www.example.com"  
                />  
                />  
              </div>  
              </div>  
            </div>  
  
            <div className="flex justify-end space-x-2 pt-2">  
            <div className="flex justify-end space-x-2 pt-2">  
              <button  
              <button  
                type="button"  
                onClick={() => setShowLinkModal(false)}  
                className="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs"  
              >  
                取消  
                取消  
              </button>  
              </button>  
              <button  
                onClick={() => {  
                onClick={() => {  
                  if (chatLinkUrl.trim()) {  
                  if (chatLinkUrl.trim()) {  
                    setShowLinkModal(false);  
                    setShowLinkModal(false);  
                  } else {  
                    alert("請填寫連結 URL！");  
                  }  
                  }  
                }}  
                }}  
                className="px-5 py-2 bg-teal-600 hover:bg-teal-500 rounded-xl text-xs font-bold text-white"  
                className="px-5 py-2 bg-teal-600 hover:bg-teal-500 rounded-xl text-xs font-bold text-white"  
              >  
                確認附加  
              </button>  
            </div>  
            </div>  
          </div>  
        </div>  
      )}  
      )}  
  
      {/* HTML 沙盒獨立安全執行環境 */}  
      {/* HTML 沙盒獨立安全執行環境 */}  
      {activeHtmlPayload && (  
        <div className="fixed inset-0 bg-black/95 backdrop-blur-md flex flex-col z-50">  
          <div className="bg-slate-900 border-b border-slate-800 p-4 flex justify-between items-center">  
            <div className="flex items-center space-x-2">  
              <span className="w-3.5 h-3.5 rounded-full bg-purple-500 animate-pulse"></span>  
              <span className="font-extrabold text-sm text-slate-200">  
                正在獨立安全執行來自 <span className="text-purple-400 font-black">{activeHtmlPayload.sender}</span> 的程式  
              </span>  
              </span>  
            </div>  
            </div>  
            <button  
              onClick={() => setActiveHtmlPayload(null)}  
              className="bg-rose-600 hover:bg-rose-500 text-white px-4 py-2 rounded-xl text-xs font-extrabold transition"  
              className="bg-rose-600 hover:bg-rose-500 text-white px-4 py-2 rounded-xl text-xs font-extrabold transition"  
            >  
            >  
              ✕ 關閉此程式沙盒  
              ✕ 關閉此程式沙盒  
            </button>  
          </div>  
  
          <div className="flex-1 bg-white">  
          <div className="flex-1 bg-white">  
            <iframe  
            <iframe  
              srcDoc={activeHtmlPayload.code}  
              srcDoc={activeHtmlPayload.code}  
              title="6A-User-Sandboxed-App"  
              title="6A-User-Sandboxed-App"  
              sandbox="allow-scripts"  
              className="w-full h-full border-none"  
              className="w-full h-full border-none"  
            />  
            />  
          </div>  
          </div>  
        </div>  
        </div>  
      )}  
  
    </div>  
    </div>  
  );  
}  
