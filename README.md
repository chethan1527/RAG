Data Ingestion:
    PyPDFLoader,PyMuPDFLoader-> Two different PDF parsing libraries
    DirectoryLpader-> Loads multiple files from a directory

model_name => all-MiniLM-L6-v2 -> model name from hugging face which helps to convert text to embeddings
Chunks:
    __init__ method:
        chunk_size=1000 ->each text chunk will be ~1000characters
        chunk_overlap=200 ->adjacent chunks share 200 characters
        sentenceTransformer(model_name) ->Loads all-MiniLM-L6-v2 model to convert text into vector embeddings
    chunk_document method:
        RecursiveCharacterTextSplitter -  splits documents by trying separators in order: \n\n, \n, space, then charactersConverts large documents into smaller, manageable chunks
        Returns list of chunk objects with page_content and metadata
    embed_chunks method:
        Extracts text from each chunk object
        model.encode() - Converts text chunks into numerical vectors (embeddings)
        Returns numpy array of shape (num_chunks, 384) - 384 is the embedding dimension for all-MiniLM-L6-v2

Embedding:
    __init__ method:
        Stores model_name ( all-MiniLM-L6-v2)
        Initializes self.model as None
        Calls _load_model() to load the model immediately

    generator_embeddings method:
        Takes a list of text strings as input
        Validates model is loaded
        Uses model.encode() to convert texts into vector embeddings

VectorSore:(Chromadb vectorstore)
    __init__ method:
        collection_name="pdf_documents" - Name of the ChromaDB collection
        persist_directory="../data/vector_store" - Where the database is saved on disk
        Initializes client and collection as None
        Calls _initialize_store() to set up the database
    _initialize_store method:
        Creates the persist directory if it doesn't exist
        PersistentClient - Creates a ChromaDB client that saves data to disk (survives restarts)
        get_or_create_collection - Gets existing collection or creates new one with metadata
        Prints collection info including document count
    add_documents method:
        Takes documents (chunks) and their embeddings as input
        Validates lengths match
        Prepares data:
            ids - Generates unique IDs using UUID 
            metadatas - Adds doc_index and content_length to existing metadata
            documents_text - Extracts page_content from each chunk
            embeddings_list - Converts numpy arrays to lists (ChromaDB format)
            Stores in ChromaDB: Calls collection.add() with all prepared data

RAGRetriever
    __init__ method:
        Takes vector_store (ChromaDB instance) and embedding_manager (for query embeddings)
        Stores both as instance variables

    retrieve method:
    Parameters:
        query - Users search question
        top_k=5 - Return top 5 most similar documents
        score_threshold=0.0 - Minimum similarity score (0-1 range)

    Query Embedding->Converts query text to 384-dimensional vector using same model (all-MiniLM-L6-v2)
    Vector Search->collection.query() finds most similar document embeddings,uses distance internally
    Result Formatting->Extracts: documents, metadatas, distances, ids from ChromaDB response,similarity_score = 1 - distance - Converts distance to similarity (higher = more similar),Creates dict with: id, similarity_score, content, metadata, distance, rank


        







