# shutter
import React, { useState, useRef, useCallback } from "react";
import { Heart, MessageCircle, Send, Bookmark, MoreHorizontal, ArrowLeft, X, Grid, UserCheck, UserPlus } from "lucide-react";

// ---------- Mock data ----------

const USERS = {
  maya: { id: "maya", name: "Maya Solano", handle: "maya.solano", avatar: "🌿", bio: "shooting film, chasing light", followers: 2840, following: 312 },
  theo: { id: "theo", name: "Theo Wren", handle: "theo.wren", avatar: "🔥", bio: "street photography / Tokyo", followers: 9120, following: 88 },
  ines: { id: "ines", name: "Inés Brava", handle: "ines.brava", avatar: "🌊", bio: "surf, salt, slow mornings", followers: 1530, following: 540 },
  kofi: { id: "kofi", name: "Kofi Aning", handle: "kofi.aning", avatar: "🌙", bio: "night markets & neon", followers: 4410, following: 201 },
  you: { id: "you", name: "You", handle: "you", avatar: "✨", bio: "", followers: 128, following: 64 },
};

const INITIAL_POSTS = [
  {
    id: "p1",
    userId: "maya",
    image: "linear-gradient(135deg,#d9a679 0%,#8a5a44 55%,#3a2a22 100%)",
    caption: "Golden hour in the orchard. Found this light and couldn't leave.",
    location: "Sonoma County",
    likedByYou: false,
    likes: 412,
    comments: [
      { id: "c1", userId: "theo", text: "this light is unreal" },
      { id: "c2", userId: "ines", text: "where is this!!" },
    ],
    timeAgo: "2h",
  },
  {
    id: "p2",
    userId: "theo",
    image: "linear-gradient(160deg,#2b2f33 0%,#15171a 60%,#000 100%)",
    caption: "3am, Shibuya. Nobody around but the neon.",
    location: "Tokyo, Japan",
    likedByYou: true,
    likes: 1208,
    comments: [{ id: "c3", userId: "kofi", text: "the mood here is everything" }],
    timeAgo: "5h",
  },
  {
    id: "p3",
    userId: "ines",
    image: "linear-gradient(135deg,#7ec8c9 0%,#2e6b6e 55%,#123638 100%)",
    caption: "Salt in my hair, sand in the camera bag. Worth it.",
    location: "Taghazout, Morocco",
    likedByYou: false,
    likes: 689,
    comments: [],
    timeAgo: "1d",
  },
  {
    id: "p4",
    userId: "kofi",
    image: "linear-gradient(140deg,#b23a48 0%,#5c1a2a 55%,#1a0a10 100%)",
    caption: "Night market smoke and red lanterns. This city never settles down.",
    location: "Accra Night Market",
    likedByYou: false,
    likes: 944,
    comments: [{ id: "c4", userId: "maya", text: "i can almost smell this photo" }],
    timeAgo: "1d",
  },
];

// ---------- Small UI atoms ----------

function Avatar({ userId, size = 36 }) {
  const u = USERS[userId];
  return (
    <div
      style={{
        width: size,
        height: size,
        borderRadius: "50%",
        background: "linear-gradient(135deg,#c9694f,#d4a857)",
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        fontSize: size * 0.5,
        flexShrink: 0,
        boxShadow: "0 0 0 2px #1a1614",
      }}
    >
      {u.avatar}
    </div>
  );
}

function FollowButton({ following, onToggle, small }) {
  return (
    <button
      onClick={onToggle}
      style={{
        display: "flex",
        alignItems: "center",
        gap: 6,
        padding: small ? "5px 12px" : "7px 18px",
        borderRadius: 999,
        fontSize: small ? 12 : 13,
        fontWeight: 600,
        fontFamily: "'Inter', sans-serif",
        border: following ? "1px solid #4a433c" : "none",
        background: following ? "transparent" : "linear-gradient(135deg,#c9694f,#b5543f)",
        color: following ? "#cfc6ba" : "#fff",
        cursor: "pointer",
        transition: "all 0.15s ease",
      }}
    >
      {following ? <UserCheck size={small ? 13 : 14} /> : <UserPlus size={small ? 13 : 14} />}
      {following ? "Following" : "Follow"}
    </button>
  );
}

// ---------- Heart burst animation ----------

function HeartBurst({ show }) {
  if (!show) return null;
  return (
    <div
      style={{
        position: "absolute",
        inset: 0,
        display: "flex",
        alignItems: "center",
        justifyContent: "center",
        pointerEvents: "none",
      }}
    >
      <Heart
        size={90}
        fill="#f5f1ea"
        color="#f5f1ea"
        style={{
          animation: "burst 0.7s ease forwards",
          filter: "drop-shadow(0 4px 14px rgba(0,0,0,0.4))",
        }}
      />
      <style>{`
        @keyframes burst {
          0% { transform: scale(0); opacity: 0; }
          15% { transform: scale(1.15); opacity: 1; }
          30% { transform: scale(0.95); opacity: 1; }
          80% { transform: scale(1); opacity: 1; }
          100% { transform: scale(1.05); opacity: 0; }
        }
      `}</style>
    </div>
  );
}

// ---------- Post card ----------

function PostCard({ post, onLike, onAddComment, onOpenProfile, following, onToggleFollow }) {
  const [showBurst, setShowBurst] = useState(false);
  const [commentText, setCommentText] = useState("");
  const [showAllComments, setShowAllComments] = useState(false);
  const lastTap = useRef(0);

  const user = USERS[post.userId];
  const isFollowing = following.has(post.userId);

  const triggerLike = useCallback(() => {
    if (!post.likedByYou) {
      onLike(post.id, true);
      setShowBurst(true);
      setTimeout(() => setShowBurst(false), 700);
    }
  }, [post, onLike]);

  const handleDoubleTap = () => {
    const now = Date.now();
    if (now - lastTap.current < 300) {
      triggerLike();
    }
    lastTap.current = now;
  };

  const visibleComments = showAllComments ? post.comments : post.comments.slice(0, 1);

  return (
    <div
      style={{
        background: "#221d19",
        borderRadius: 18,
        overflow: "hidden",
        marginBottom: 28,
        border: "1px solid #332b25",
      }}
    >
      {/* header */}
      <div style={{ display: "flex", alignItems: "center", padding: "14px 16px", gap: 12 }}>
        <button onClick={() => onOpenProfile(post.userId)} style={{ background: "none", border: "none", padding: 0, cursor: "pointer", display: "flex" }}>
          <Avatar userId={post.userId} />
        </button>
        <div style={{ flex: 1, minWidth: 0 }}>
          <button
            onClick={() => onOpenProfile(post.userId)}
            style={{ background: "none", border: "none", padding: 0, cursor: "pointer", display: "block", textAlign: "left" }}
          >
            <div style={{ fontFamily: "'Fraunces', serif", fontWeight: 600, fontSize: 15, color: "#f5f1ea" }}>{user.name}</div>
          </button>
          <div style={{ fontSize: 12, color: "#8a8076" }}>
            {post.location} · {post.timeAgo}
          </div>
        </div>
        {post.userId !== "you" && (
          <FollowButton small following={isFollowing} onToggle={() => onToggleFollow(post.userId)} />
        )}
        <MoreHorizontal size={18} color="#8a8076" />
      </div>

      {/* image */}
      <div
        onClick={handleDoubleTap}
        style={{
          position: "relative",
          width: "100%",
          aspectRatio: "1/1",
          background: post.image,
          cursor: "pointer",
        }}
      >
        <div
          style={{
            position: "absolute",
            inset: 0,
            background:
              "repeating-linear-gradient(0deg, rgba(0,0,0,0.04) 0px, transparent 1px, transparent 2px)",
            mixBlendMode: "overlay",
            opacity: 0.5,
          }}
        />
        <HeartBurst show={showBurst} />
      </div>

      {/* actions */}
      <div style={{ display: "flex", alignItems: "center", padding: "12px 16px 4px", gap: 16 }}>
        <button
          onClick={() => onLike(post.id, !post.likedByYou)}
          style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}
        >
          <Heart
            size={24}
            color={post.likedByYou ? "#c9694f" : "#cfc6ba"}
            fill={post.likedByYou ? "#c9694f" : "none"}
            style={{ transition: "transform 0.15s ease" }}
          />
        </button>
        <button style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}>
          <MessageCircle size={23} color="#cfc6ba" />
        </button>
        <button style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}>
          <Send size={21} color="#cfc6ba" />
        </button>
        <div style={{ flex: 1 }} />
        <button style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}>
          <Bookmark size={21} color="#cfc6ba" />
        </button>
      </div>

      {/* likes count */}
      <div style={{ padding: "6px 16px 0", fontSize: 13.5, color: "#f5f1ea", fontWeight: 600, fontFamily: "'Inter', sans-serif" }}>
        {post.likes.toLocaleString()} likes
      </div>

      {/* caption */}
      <div style={{ padding: "6px 16px 0", fontSize: 13.5, color: "#e8e2d8", lineHeight: 1.5 }}>
        <span style={{ fontWeight: 700, marginRight: 6 }}>{user.handle}</span>
        {post.caption}
      </div>

      {/* comments */}
      <div style={{ padding: "8px 16px 0" }}>
        {post.comments.length > 1 && !showAllComments && (
          <button
            onClick={() => setShowAllComments(true)}
            style={{ background: "none", border: "none", color: "#8a8076", fontSize: 13, cursor: "pointer", padding: 0, marginBottom: 4 }}
          >
            View all {post.comments.length} comments
          </button>
        )}
        {visibleComments.map((c) => (
          <div key={c.id} style={{ fontSize: 13.5, color: "#e8e2d8", marginBottom: 4 }}>
            <span style={{ fontWeight: 700, marginRight: 6 }}>{USERS[c.userId].handle}</span>
            {c.text}
          </div>
        ))}
      </div>

      {/* add comment */}
      <form
        onSubmit={(e) => {
          e.preventDefault();
          if (commentText.trim()) {
            onAddComment(post.id, commentText.trim());
            setCommentText("");
            setShowAllComments(true);
          }
        }}
        style={{
          display: "flex",
          alignItems: "center",
          gap: 10,
          padding: "10px 16px 16px",
          borderTop: "1px solid #2c2521",
          marginTop: 10,
        }}
      >
        <input
          value={commentText}
          onChange={(e) => setCommentText(e.target.value)}
          placeholder="Add a comment..."
          style={{
            flex: 1,
            background: "transparent",
            border: "none",
            outline: "none",
            color: "#f5f1ea",
            fontSize: 13.5,
            fontFamily: "'Inter', sans-serif",
          }}
        />
        {commentText.trim() && (
          <button
            type="submit"
            style={{ background: "none", border: "none", color: "#c9694f", fontWeight: 700, fontSize: 13, cursor: "pointer" }}
          >
            Post
          </button>
        )}
      </form>
    </div>
  );
}

// ---------- Profile view ----------

function ProfileView({ userId, posts, onBack, following, onToggleFollow, isOwnProfile }) {
  const user = USERS[userId];
  const userPosts = posts.filter((p) => p.userId === userId);
  const isFollowing = following.has(userId);

  return (
    <div>
      <div style={{ display: "flex", alignItems: "center", gap: 14, padding: "16px 16px 6px" }}>
        <button onClick={onBack} style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}>
          <ArrowLeft size={22} color="#f5f1ea" />
        </button>
        <div style={{ fontFamily: "'Fraunces', serif", fontWeight: 600, fontSize: 17, color: "#f5f1ea" }}>{user.handle}</div>
      </div>

      <div style={{ padding: "18px 20px 8px", display: "flex", gap: 22, alignItems: "center" }}>
        <Avatar userId={userId} size={84} />
        <div style={{ flex: 1, display: "flex", justifyContent: "space-around", textAlign: "center" }}>
          <div>
            <div style={{ fontSize: 17, fontWeight: 700, color: "#f5f1ea" }}>{userPosts.length}</div>
            <div style={{ fontSize: 12, color: "#8a8076" }}>posts</div>
          </div>
          <div>
            <div style={{ fontSize: 17, fontWeight: 700, color: "#f5f1ea" }}>{user.followers.toLocaleString()}</div>
            <div style={{ fontSize: 12, color: "#8a8076" }}>followers</div>
          </div>
          <div>
            <div style={{ fontSize: 17, fontWeight: 700, color: "#f5f1ea" }}>{user.following}</div>
            <div style={{ fontSize: 12, color: "#8a8076" }}>following</div>
          </div>
        </div>
      </div>

      <div style={{ padding: "4px 20px" }}>
        <div style={{ fontWeight: 700, fontSize: 14, color: "#f5f1ea", fontFamily: "'Fraunces', serif" }}>{user.name}</div>
        {user.bio && <div style={{ fontSize: 13, color: "#cfc6ba", marginTop: 2 }}>{user.bio}</div>}
      </div>

      {!isOwnProfile && (
        <div style={{ padding: "14px 20px 6px" }}>
          <div style={{ width: "100%" }}>
            <FollowButton following={isFollowing} onToggle={() => onToggleFollow(userId)} />
          </div>
        </div>
      )}

      <div style={{ display: "flex", alignItems: "center", gap: 6, padding: "18px 20px 10px", borderTop: "1px solid #2c2521", marginTop: 14 }}>
        <Grid size={15} color="#8a8076" />
        <span style={{ fontSize: 12, color: "#8a8076", fontWeight: 600, letterSpacing: 0.5 }}>POSTS</span>
      </div>

      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 3, padding: "0 0 20px" }}>
        {userPosts.map((p) => (
          <div key={p.id} style={{ aspectRatio: "1/1", background: p.image }} />
        ))}
        {userPosts.length === 0 && (
          <div style={{ gridColumn: "1/-1", textAlign: "center", padding: "40px 20px", color: "#8a8076", fontSize: 13 }}>
            No posts yet.
          </div>
        )}
      </div>
    </div>
  );
}

// ---------- Main app ----------

export default function App() {
  const [posts, setPosts] = useState(INITIAL_POSTS);
  const [view, setView] = useState({ screen: "feed" }); // feed | profile
  const [following, setFollowing] = useState(new Set(["ines"]));

  const handleLike = (postId, liked) => {
    setPosts((prev) =>
      prev.map((p) =>
        p.id === postId
          ? { ...p, likedByYou: liked, likes: p.likes + (liked ? 1 : -1) }
          : p
      )
    );
  };

  const handleAddComment = (postId, text) => {
    setPosts((prev) =>
      prev.map((p) =>
        p.id === postId
          ? { ...p, comments: [...p.comments, { id: `c${Date.now()}`, userId: "you", text }] }
          : p
      )
    );
  };

  const handleToggleFollow = (userId) => {
    setFollowing((prev) => {
      const next = new Set(prev);
      if (next.has(userId)) next.delete(userId);
      else next.add(userId);
      return next;
    });
  };

  const openProfile = (userId) => setView({ screen: "profile", userId });
  const backToFeed = () => setView({ screen: "feed" });

  return (
    <div
      style={{
        minHeight: "100vh",
        background: "#1a1614",
        fontFamily: "'Inter', sans-serif",
        color: "#f5f1ea",
      }}
    >
      <link
        rel="stylesheet"
        href="https://fonts.googleapis.com/css2?family=Fraunces:wght@500;600;700&family=Inter:wght@400;500;600;700&display=swap"
      />

      {/* top bar */}
      <div
        style={{
          position: "sticky",
          top: 0,
          zIndex: 10,
          background: "rgba(26,22,20,0.92)",
          backdropFilter: "blur(8px)",
          borderBottom: "1px solid #2c2521",
          padding: "16px 20px",
          display: "flex",
          alignItems: "center",
          justifyContent: "space-between",
        }}
      >
        <div
          style={{
            fontFamily: "'Fraunces', serif",
            fontWeight: 700,
            fontSize: 22,
            letterSpacing: "-0.02em",
            color: "#f5f1ea",
          }}
        >
          shutter<span style={{ color: "#c9694f" }}>.</span>
        </div>
        <button
          onClick={() => openProfile("you")}
          style={{ background: "none", border: "none", cursor: "pointer", display: "flex" }}
        >
          <Avatar userId="you" size={32} />
        </button>
      </div>

      <div style={{ maxWidth: 480, margin: "0 auto", padding: view.screen === "feed" ? "20px 16px 60px" : "0 0 60px" }}>
        {view.screen === "feed" && (
          <>
            {posts.map((post) => (
              <PostCard
                key={post.id}
                post={post}
                onLike={handleLike}
                onAddComment={handleAddComment}
                onOpenProfile={openProfile}
                following={following}
                onToggleFollow={handleToggleFollow}
              />
            ))}
            <div style={{ textAlign: "center", color: "#5e564e", fontSize: 12.5, padding: "10px 0 30px" }}>
              You're all caught up · shutter
            </div>
          </>
        )}

        {view.screen === "profile" && (
          <ProfileView
            userId={view.userId}
            posts={posts}
            onBack={backToFeed}
            following={following}
            onToggleFollow={handleToggleFollow}
            isOwnProfile={view.userId === "you"}
          />
        )}
      </div>
    </div>
  );
}
