import { useEffect, useState } from "react";
import { getAuth, signInWithPopup, GoogleAuthProvider, signOut } from "firebase/auth";
import { getFirestore, collection, getDocs } from "firebase/firestore";
import { initializeApp } from "firebase/app";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

export default function Home() {
  const [user, setUser] = useState(null);
  const [mangaList, setMangaList] = useState([]);

  useEffect(() => {
    auth.onAuthStateChanged(setUser);
    fetchManga();
  }, []);

  const login = async () => {
    const provider = new GoogleAuthProvider();
    await signInWithPopup(auth, provider);
  };

  const logout = async () => {
    await signOut(auth);
  };

  const fetchManga = async () => {
    const querySnapshot = await getDocs(collection(db, "manga"));
    setMangaList(querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })));
  };

  return (
    <div className="p-6">
      <div className="flex justify-between items-center mb-4">
        <h1 className="text-2xl font-bold">Manga Website</h1>
        {user ? (
          <Button onClick={logout}>Logout</Button>
        ) : (
          <Button onClick={login}>Login with Google</Button>
        )}
      </div>
      <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
        {mangaList.map((manga) => (
          <Card key={manga.id} className="p-2">
            <CardContent>
              <img src={manga.cover} alt={manga.title} className="w-full h-48 object-cover rounded" />
              <h2 className="text-lg font-semibold mt-2">{manga.title}</h2>
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
